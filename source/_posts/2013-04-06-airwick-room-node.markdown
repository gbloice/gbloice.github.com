---
layout: post
title: "Airwick room node"
date: 2013-04-06 23:24:22 +0000
comments: true
categories: [jeenode]
published: true
---

To create a room sensor node with the JNu, I've attached a [AM2302][1] (also known as a DHT22) temperature and humidity sensor and modified the radio blip sketch to communicate with the sensor and return the data in the radio blip payload.

<!--more-->

The sensor has 4 pins, 3 of which are used; Vdd, Data, and Ground, and these are connected to the JNu +, R and G pins of the single port on the JNu.  A 10K&#x2126; pullup resistor is connected between the Vdd and Data pins.

The sketch used is in the [Airwick][2] github repo and an Arduino [DHT22 library][3].  The sketch differs from the normal radio blip by calling the read22 routine to start a conversion attempt and then if successful, reading the resulting temperature and humidity values.  These floating point values are then multiplied by 10 and converted to integers for transmission over the radio, along with the status of the sensor conversion attempt.  The resulting payload can then be observed using the JNUSB:

```
OK 22 93 1 0 0 2 117 118 224 0 243 1 0
OK 22 94 1 0 0 2 117 118 225 0 245 1 0
OK 22 95 1 0 0 2 117 118 225 0 242 1 0
OK 22 96 1 0 0 2 117 118 0 0 0 0 255
OK 22 97 1 0 0 2 117 118 226 0 237 1 0
OK 22 98 1 0 0 2 117 118 228 0 236 1 0
```

The first part of the data is the same as the standard radio blip, the values at the back end are the temperature and the humidity (16 bit little endian values) and the sensor conversion status (single byte).  In the case of the last entry `OK 22 98 1 0 0 2 117 118 228 0 236 1 0` the value `228 0` is the temperature (22.8°C), the value `236 1` is the humidity (49.2%) and the status is `0` (OK).  Note the one entry where the temperature and humidity values are 0 and the status is 255 which indicates a sensor conversion failure.  These occur occasionally for unknown reasons.


  [1]: http://www.adafruit.com/products/385
  [2]: https://github.com/gbloice/AirWick
  [3]: http://playground.arduino.cc/Main/DHTLib