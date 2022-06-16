---
layout: post
title:  "JellingStone rewrite - a new wire procotol"
date:   2022-05-25 16:27:18 +0100
categories: misc
---

JellingStone is the software for ESP32-devices. It utilizes Bluetooth for scanning (i.e. detecting) and emitting beacons. 
Results are transmitted over MQTT and WLAN. It is based on [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html).

Recently, a rewrite was needed, and we moved to a byte-oriented wire-protocol for reports.
It is already implemented in JellingStone but changes to the StoneAggregator are yet to be done.

<!--break-->

### Changes in ESP-IDF

Over the past years, ESP-IDF has undergone significant changes. Components such as MQTT are integrated and
do no longer require 3rd-party dependencies. The build-system moved from Makefiles to CMake and the toolchain addresses
more modern Python versions. In result, only a build-server based on Debian oldstable is able to 
build the software correctly, still. This motivated revisiting and rewriting large parts of the application.

### Application redesign

When staring JellingStone back in 2017 / 2018, it was designed as a research prototype. Not knowing exactly what was needed,
we instead focused on exploring what was possible. Our motto was simplicity over optimization. For scan-reports we went 
for a simple protocol. Interesting data such as MAC-addresses, UUIDs, min/max/avg signal strength was encoded in JSON. 
Almost 100 byte of data were needed per detected beacon. Of course, this can be reduced by compression. Still, 
such a report includes a lot of data that is neither processed and analyzed. 
Growing and compressing a JSON string consumes memory, whereas only 160 KiB of RAM are dynamically available. 
In addition, encrypting, fragmenting and transmitting larger MQTT-payloads puts significant stress on ESP32 devices. 

### Avoiding Network Fragmentation

When trying to avoid fragmentation on lower network layers, it is important that one complete MQTT message can
be put into one Ethernet frame. Realistically, Ethernet can transmit at least 1280 byte per frame ([RFC2460](https://datatracker.ietf.org/doc/html/rfc2460#section-5)). 
Typical TCP- and IP headers reduce the available capacity by 60 byte (IPv6-Header: 40 byte, TCP-Header 20 byte).
Using TLS encryption further reduces the capacity by ~ [40-60 byte](https://netsekure.org/2010/03/tls-overhead/) due to padding
and message authentication codes. Of course, this depends on the ciphers in use.
In total, this leaves about 1280 - 60 - 60 = 1160 byte for MQTT payloads. The MQTT-header is relatively large, 
because the topic name is included in every message. For instance, 37 byte are needed to encode a name
of a JellingStone scan-report topic. Hence, roughly about **1100 byte** are appropriate to be used for actual data.

### Wire-protocol Design 

Having about 1 KiB available for data motivates a relatively compact encoding scheme. However, various BLE standard
use relatively large identifiers to avoid collisions between beacons of different networks and standards.
 
* AltBeacon: [20 byte Beacon ID](https://github.com/AltBeacon/spec)
* Eddystone UID: [16 byte Beacon ID](https://github.com/google/eddystone/tree/master/eddystone-uid) (10 byte namespace, 6 byte instance)
* Eddystone EID: [8 byte Ephemeral ID](https://github.com/google/eddystone/tree/master/eddystone-eid) 

In essence, one needs to carry up to 8-20 byte of ID data per Beacon. However, most parts are rather redundant. 
For Eddystone UID, the 10 byte namespace is not of interest, because it refers to the beacon-network (e.g. dev.fieldtracks.org).
In addition, an instance ID of 6 byte is relatively large for fieldtracks. It is unrealistic to track more
than 65536 people during a field exercise training. Hence, the instance ID potentially needs just 1 or 2 byte 
depending on number of participants and the numbering scheme. Of course, one needs an additional byte to encode the 
beacon-type and the detected RSSI each.

* Using EID for privacy requires 10 byte per beacon: *~100 scan-results per message*
* Using UID and requires just 4 byte per beacon:  *~256 scan-results per message*

### Protocol specification

#### Header

| Byte     | Description                                                                                                                                                                                     |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0        | `0x01` (Version), reserved: `0x7B` for JSON Payloads                                                                                                                                            |
| 1-4      | Report-ID / 32-Bit timestamp (seconds since Unix-epoch), Big-Endian / Network Byte Order                                                                                                        |
| 5        | Message sequence number (signed), unique per report, starts at `0x01` in each report, incremented per message, a negative number marks final one, e.g. `0xFF` (-1), if there's just one message |
| 6        | unsigned, number of beacon data segments in this message (report has more, if and only if there are more messages)                                                                              |
| 7 - 1100 | 0...255 (up to 255) segments of beacon data                                                                                                                                                     |

#### Beacon Data segment

| Byte   | Description                                                                                                                                                                                                                                                                                                                                                                     |
|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0      | Type and length of Beacon ID in byte. <br /> `0x14` AltBeacon <br /> `0x08` Eddystone EID <br /> `0x10` Eddystone UID <br /> `0x06` Eddystone UID, namespace matches configured one <br > `0x01` Eddystone UID, namespace matches configured one, instance <= 255 (i.e. 1 byte) <br /> `0x02` Eddystone UID, namespace matches configured one, instance <= 65536 (i.e. 2 bytes) |
| 1      | signed, detected RSSI in dBm + 100 (i.e. -228 dBm to 27 dBm) encoded as -128 to 127                                                                                                                                                                                                                                                                                             |
| 2 - 21 | Beacon ID, variable length                                                                                                                                                                                                                                                                                                                                                      |


#### Remarks
* Beacon data is encoded on a type-value basis, whereas the byte value of the type corresponds to the length of the ID-value
* The reporting stone (i.e. sender address) is included in the topic-name. It is part of the MQTT-header, hence.
* Using a 32-bit timestamp is motivated by `time_t` having 32 bit in ESP-IDF. Due to a year 2036-problem that may change in later versions of the protocol. 

### Conclusion and ideas

Re-designing the wire protocol allows to transmit beacons more efficiently. Depending on the privacy-requirements,
up to 255 scan results can be transmitted in a MQTT frame. That's a huge improvement over the existing JSON based protocol.

Intuitively, receiving up to 255 beacons or more during a scan period of 8 seconds doesn't seem to be completely absurd in the first place.
However, guessing somewhat realistic numbers is out-of-scope for this article.

Nevertheless, one could think about using this wire-protocol with LoRaWAN instead of MQTT. This won't work. A typical LoRa
frame has only 37 byte (51 byte - 13 byte for LoRaWAN-header) - up to 3 beacons could be transmitted at once. Duty cycle limitations suggest a scan-interval of 2 minutes at least.
Hence, the potential is quite limited. Furthermore, it is open which data is actually useful when using LoRaWAN, i.e. long-distance
links. Do we need a timestamp, still? Do we need RSSI-data? How many LoRa-speakers are realistic? 

Happy Hacking ;-).
