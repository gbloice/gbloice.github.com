---
layout: post
title: "Intro to JeeNodes"
date: 2013-03-31 13:53
comments: true
categories: [jeenode, arduino, buspirate, avrdude]
published: true
---

To make a room sensor I'm going to use a [JeeNode Micro][jlu] (JNu) reporting back to a [JeeNode USB][jlusb] (JNUSB) running on my home server.

<!--more-->

After receiving the items from the [JeeLabs shop][jlshop] I connected them up to test on the bench.  I soldered the aerial wires to each item, cutting them to 82mm for the 868MHz radios.

The JNUSB comes ready flashed with the [RFM12Demo][jl-rf12demo] sketch from the [JeeLib][jlib] examples directory so simply requires plugging in to a USB port and then opening a terminal to the appropriate COM port created by the USB drivers.  As I'm testing on a Windows system, I looked in Device Manager | Ports to see what pseudo com port number was, and then opened [PuTTY][putty] for that com port.  Set the baud rate to 57600 and hit enter to see the standard RF12demo response:

 [jlu]:   	http://jeelabs.net/projects/hardware/wiki/JeeNode_Micro
 [jlusb]:	http://jeelabs.net/projects/hardware/wiki/JeeNode_USB
 [jlshop]:	http://jeelabs.com/
 [jlib]:	https://github.com/jcw/jeelib
 [jl-rf12demo]:	https://github.com/jcw/jeelib/tree/master/examples/RF12/RF12demo
 [putty]:	http://www.chiark.greenend.org.uk/~sgtatham/putty/
 
```
[RF12demo.7] A i1 g5 @ 868 MHz

Available commands:
  <nn> i     - set node ID (standard node ids are 1..26)
               (or enter an uppercase 'A'..'Z' to set id)
  <n> b      - set MHz band (4 = 433, 8 = 868, 9 = 915)
  <nnn> g    - set network group (RFM12 only allows 212, 0 = any)
  <n> c      - set collect mode (advanced, normally 0)
  t          - broadcast max-size test packet, with ack
  ...,<nn> a - send data packet to node <nn>, with ack
  ...,<nn> s - send data packet to node <nn>, no ack
  <n> l      - turn activity LED on PB1 on or off
  <n> q      - set quiet mode (1 = don't report bad packets)
Remote control commands:
  <hchi>,<hclo>,<addr>,<cmd> f     - FS20 command (868 MHz)
  <addr>,<dev>,<on> k              - KAKU command (433 MHz)
Current configuration:
 A i1 g5 @ 868 MHz
```

Note that I've issued the group command (5g) to set the group to the group used by the JNu in the next section.

For the JNu, things are a little more complicated as it doesn't appear to come flashed with a sketch, nor does it have a bootloader.  JeeLabs has a guide to compiling and flashing the micro ([Pt1][jlu-1], [Pt2][jlu-2], [Pt3][jlu-3] and [Pt4][jlu-4]) but I had some issues completing the flash as I'm using a [Bus Pirate][bp](BP) as my ISP programmer and I couldn't persuade the IDE to upload the hex file.  To workaround this I followed the instructions from Pt2 and manually fired up avrdude to upload the sketch using the BP.  After uploading the sketch, the terminal connected to the JNUSB started showing the "blip" messages from the JNu, with a new message every 60 seconds:

```
OK 22 1 0 0 0 1 118 0
OK 22 2 0 0 0 1 117 118
OK 22 3 0 0 0 1 117 118
```

The values in the string are the reception OK flag, the node ID (22), the 32 bit message count which increments every message, and the power supply voltage before and after the transmission.  Note the first message doesn't have an after voltage as 0 as this lags by 1 transmission.  The voltage value is scaled to 1.0V = 0, 6.0V = 250, so 117 is (117 / 50 + 1.0) 3.34V, which is the power supply from the BP.  I also set the JNUSB to quiet mode (1q) so random noise isn't reported to the terminal.

 [jlu-1]:	http://jeelabs.org/2013/03/03/programming-the-jnu-v3-part-1/
 [jlu-2]:	http://jeelabs.org/2013/03/07/programming-the-jn%C2%B5-v3-part-2/
 [jlu-3]:	http://jeelabs.org/2013/03/20/programming-the-jnu-again/
 [jlu-4]:	http://jeelabs.org/2013/03/21/programming-the-jn%C2%B5-at-last/
 [bp]:		http://dangerousprototypes.com/docs/Bus_Pirate