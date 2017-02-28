---
author: yamila
comments: true
date: 2014-12-17 15:56:26+00:00
slug: arduino-from-command-line
title: Arduino from command line
wordpress_id: 649
categories:
- Open source
- Tutorials
tags:
- arduino
- bash
- linux
- open source
- piweek
- vim
---

We are used to work in the [Arduino IDE](http://arduino.cc/en/Main/Software#toc3); from this IDE it's possible to create a sketch, compile it, upload to any Arduino and also it has a serial monitor to debugg.

For me it's very useful, but I'm more confortable in the command line...
<!-- more -->

This week I'm participating in the [VII PIWeek](http://piweek.com), building (we'll see) a _thing_ to drive a drone thanks to sensors located in the arm.

I have to make many tests and I prefer to use _my_ [Vim](http://www.vim.org/) and _my_ [Sakura](https://launchpad.net/sakura) terminal. Now it's possible!!

**Compile/verify**



    cd /my/sketch
    /path/to/arduino --verify sketch.ino




**Upload**



    /path/to/arduino --upload sketch.ino




In both cases _verify_ and _upload_, the expected output is something like:



    Sketch uses 16.106 bytes (56%) of program storage space. Maximum is 28.672 bytes.
    Global variables use 950 bytes (37%) of dynamic memory, leaving 1.610 bytes for local variables. Maximum is 2.560 bytes.




**Serial monitor**
I found [picocom](https://code.google.com/p/picocom/), a very easy tool to see what's happening in the serial port. Once the sketch is uploaded, open a new terminal and run:



    picocom -b 9600 /dev/ttyACM0




whith the bauds and the device. And that's all!!
