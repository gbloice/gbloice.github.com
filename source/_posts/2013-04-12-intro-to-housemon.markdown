---
layout: post
title: "Intro to HouseMon"
date: 2013-04-12 23:39
comments: true
categories: [jeenode, housemon, angularjs]
published: true
---

Having got a room sensor up and running on the bench, I now want to log the values somewhere.  I'm still searching for a Home Automation server that ticks all the boxes, so for now I'll use HouseMon from Jeelabs.  The birds-eye view of how it works is [here][1], and the source can be found on GitHub [here][2].

  [1]: http://jeelabs.org/tag/housemon/
  [2]: https://github.com/jcw/housemon

<!--more-->

On my Linux server I cloned the github repo, installed the prerequisites and ran it up to check it worked.  To customise HouseMon for my room node I created a local nodeMap, briqs/nodeMap-local.coffee, that maps the RF12 group and node ID (5 and 22 respectively) to the airwickNode driver, and also names the current location of the sensor.  I then created a new driver, drivers/airwickNode.coffee, that describes how to extract the data from the rf12 packet transmitted by the airwick node.

The local nodeMap is relatively self-explanatory:

``` coffeescript nodeMap-local.coffee
exports.rf12nodes =
  5:
    22: 'airwickNode'

exports.locations =
  'RF12:5:22': title: 'computer room'
```

but the driver is more interesting:

``` coffeescript airwickNode.coffee
module.exports =

  announcer: 11

  descriptions:
    ping:
      title: 'Ping count'
      min: 0
    age:
      title: 'Estimated age'
      unit: 'days'
      min: 0
    vpre:
      title: 'Vcc before send'
      unit: 'V'
      min: 0
      factor: 2
      scale: 2
    vpost:
      title: 'Vcc after send'
      unit: 'V'
      min: 0
      factor: 2
      scale: 2
    temp:
      title: 'Temperature'
      unit: '°C'
      scale: 1
      min: -50
      max: 50
    humi:
      title: 'Relative humidity'
      unit: '%'
      scale: 1
      min: 0
      max: 100

  feed: 'rf12.packet'

  decode: (raw, cb) ->
    count = raw.readUInt32LE(1)
    t = raw.readUInt16LE(8, true)
    h = raw.readUInt16LE(10, true)
    result =
      ping: count
      age: count / (86400 / 60) | 0
      temp: t
      humi: h
      vpre: 50 + raw[6]
      vpost: 50 + raw[7]
    cb result
```

Each field in the packet is given a description, and where appropriate some units, min and max values and scale and factor values.  Scale and factor are used to convert the fixed point integer value from the driver into a decimal value for display.  For example, a temperature value of 226 is 22.6°C, so this requires a scale of 1 to divide it by 10.  The Vcc values require both scale and factor and an offset to turn a value of 117 into 3.34V.  The offset is supplied by the decode function where 50 is added to the raw packet value, giving 167, a factor of 2 gives 334, and a scale of 2 divides it by 100 to give 3.34.

With these additions, the Housemon instance is then configured from the admin page:

{% img /images/HouseMon-Admin.PNG HouseMon - Admin %}

Note the items I've installed, the only item that required a configuration setting was the rf12demo briq and that requires the serial port that I've attached the JeeNode to, in my case ttyUSB0.

The following images show the HouseMon Readings, Status and Graphs pages, with the temperature selected on the Graph and the timespan set to 24 hours.

{% img /images/HouseMon-Readings.PNG HouseMon - Readings %}

{% img /images/HouseMon-Status.PNG HouseMon - Status %}

{% img /images/HouseMon-Graph.PNG HouseMon - Graphs %}


