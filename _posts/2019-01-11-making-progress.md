---
layout: post
title:  "Making progress"
date:   2019-02-25 20:14:18 +0200
categories: status
---
It has been a while since the project started and we have made some progress in development. Let us have short look at our current codebase:

- [JellingStone](https://github.com/FieldTracks/JellingStone) is our ESP32 firmware. It supports Bluetooth Low Energy (BLE) scanning and beaconing and is almost complete.
Sensor data is transported using Wifi and MQTT.
- [StoneAggregator](https://github.com/FieldTracks/StoneAggregator) is a python script to process MQTT-data originating from JellingStone. It aggregates sensor data and stores it using retained MQTT messages. By that, clients receive all data directly after connecting.
- [fieldmon](https://github.com/FieldTracks/fieldmon) is a Angular based web application to display sensor data and manage stones. It is probably the most complex component of Fieldtracks. Unlike traccar or owntracks, fieldmon performs localization using a particle simulation. It is under heavy development.
- [StoneSimulator](https://github.com/FieldTracks/StoneSimulator) is a rather small python script for testing. It generates sensor data and publishes it via MQTT.

Besides of that, there is [mqtt2msql](https://github.com/FieldTracks/mqtt2sql) to log all sensor data to a MySQL database and [ansible-envs](https://github.com/FieldTracks/ansible-envs) to instantiate environments for integration testing. Flashtool - mqtt client for flashing esp32 devices using a Raspberry Pi is in early development and will be published soon.
