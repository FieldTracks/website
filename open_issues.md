---
layout: page
title: Open Issues
permalink: /open_issues/
---
This page lists open questions and issues related to Fieldtracks. 
They are more or less independent of the actual implementation, but relevant. 

Some fit for blog posts, while others may suit as a lab- or seminar topic (undergraduate level).
Feel free to contacts us if you're interested in testing or research. 

### 1. What parameters to choose?
On the esp32 it is possible to adjust: scan window, scan interval, collection interval, 
beacon interval, transmission power. What is a good choice?
How long does it take to detect a device among N others in a room?  

### 2. Creating a distributed MQTT middleware?
Sometimes field exercises have different hotpots (scenarios, points-of-interest) 
scattered over a large area. Network access may be limited (GPRS, G.SHDSL), 
but data could be collected locally (i.e. Raspberry Pi). 
While exact real-time data is very helpful for staff nearby, remote command posts 
have more relaxed requirements on resolution and real-time.
Can a middleware based on MQTT and anycast help? How to deal with configuration?

### 3. Using Lora / LoraWAN for data transmission?
Some scenarios cover a large area without internet connectivity (i.e. forest, valley) 
and no central points of interest. Here, Lora / or Lorawan may be used to 
transmit data. Having only a very small data rate, this requires adjustments
to the wire protocol and a distributed lora wan backend (i.e. without internet connectivity).
How can Lora or LoraWAN be used in such a scenario?

### 4. Meshing using esp32?
Instead of using an external wifi infrastructure, esp32 device can span their own mesh 
(wifi, BLE?). This puts more load on the radios and radio channels, but eliminates the
need for an additional infrastructure. What is a good way to create a mesh? 
What is the impact on scanning and beaconing? Do BLE-meshes limit the impact?

### 5. Distance and mapping?

In its basic form, fieldtracks will just look at the rssi-vector for positioning. However,
more sophisticated models (machine learning, static) allow better positioning . 
Have a look at SOME for instance:

*SOMBE: Self-Organizing Map for Unstructured and Non-Coordinated iBeacon
Constellations. Duc V. Le ; Wouter van Kleunen; Thuong C. Nguyen; Nirvana Meratnia 
and Paul Havinga*

Which models can help you deriving relative beacon positions?
Can machine learning deal with rapidly changing radio characteristics during a field exercise?
(I.e. metal rescue equipment shielding transmitters in small tunnels).

### 6. GPS support for positioning?

Cheap GPS modules can help with positioning outdoor beacons. 
What modules are suitable? How to integrate these modules?

### 7. Using Wi-Fi and other sensors in addition?

Smartphones provide different sensors (i.e. gait counter, barometer). In addition, the Wi-Fi signal can 
be used for positioning. Using this information is challenging and may require active scanning 
(i.e. Wi-Fi using esp8266 modules in addition). 
What are promising sources of information? How to integrate these sources?
