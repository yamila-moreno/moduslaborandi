---
author: yamila
comments: true
date: 2014-12-23 08:31:18+00:00
slug: vii-piweek-jannie-5
title: 'VII PiWeek: Jannie 5'
wordpress_id: 642
categories:
- General
---

_How many times have you wanted or desperately needed an assassin drone?_ With this question, [Alex](http://twitter.com/lekum) and I came to the _maketplace_ of the [VII PIWeek](http://piweek.com).

> Jannie-5 is a kickass and assistant drone driven with a bionic arm and the voice of its owner. It is extremely dangerous when
> it gets hungry, but very useful when you are fighting kaijus or zombie nazis.


TL;DR: You can go directly to the videos, or read the [short version](http://www.kaleidos.net/blog/824/πweek-jannie-5/) of this project.
<!-- more -->

Oh, wait, maybe you don't know what is PIWeek; well [PIWeek](http://piweek.com/) stands for _Personal Innovation Week_ and consists in a whole week where participants only work in offside projects, with two rules:


* You have to use and develop _Open Source_
* You have to show something functional by friday (no slides or ideas but actual code working)


Would you like to join us in the VIII PIWeek? Send an email to [hello@kaleidos.net](mailto:hello@kaleidos.net). Well, the main idea for this PIWeek was fiddle with Arduino and the perfect excuse to buy a drone:


* The Arduino was the [Yún](http://arduino.cc/en/Main/ArduinoBoardYun?from=Products.ArduinoYUN) model
* The Drone was [Parrot ARdrone 2 Elite Edition](http://ardrone2.parrot.com/) model


We chose the ARdrone because it's driven with WiFi, and the Yún because it combines the power of Arduino pins with a simple Linux and WiFi access. The final trigger was that we found this amazing [python ardrone](https://github.com/venthur/python-ardrone/) library so we had the main structure.

We started with a simple scheme:

<figure>
  <img src="/images/2014/12/scheme.jpg" width=""
       alt="Simple scheme for world domination" />
  <figcaption>Simple scheme for world domination</figcaption>
</figure>


* Initially, we thought of a robotic arm with sensors, leds, music, and some ligth weapons.
* The information of the sensors would reach the Arduino
* This Arduino connected to a Linux with WiFi
* Using a Python library in a clean and perfect way
* Ocassionally, some gummy bears would rained


The scheme was almost perfect!! It was only a matter of connecting the components...

1. The drone

The first thing was, of course, download the [official application](https://play.google.com/store/apps/details?id=com.parrot.freeflight) to control the drone. Simple and intuitive panel and a very fluid flight. It's really awsome.

2. Libardrone

Then, we had to be sure about other third party component. This library exposes some actions like _takeoff_, _spin to the right_ or _land_ and translates them to [AT Commands](http://en.wikipedia.org/wiki/Hayes_command_set). Besides, the library creates and manages the socket for UDP connection. This is where magic begins: have my action reached the drone? who knows!!

(this blog is also UDP: as my dear readers aren't used to commenting, I don't really know if you are here ^_^)

The original library was written by [Bastian Ventur](https://github.com/venthur/python-ardrone). It implements lots of functionalities, more than we needed for this first version, so we took it and removed some parts; the [result](https://github.com/yamila-moreno/jannie-5/blob/master/src/libardrone.py) was lighter and more deterministic (perfect for our [_mvp_](http://en.wikipedia.org/wiki/Minimum_viable_product)).

We connected my computer to the ARDrone WiFi and tried this [example](https://github.com/yamila-moreno/jannie-5/blob/master/examples/libardrone-example.py) to check everything was ok. And it was! We also made the same test from the Linux in the Yún.

3. Bottle

We needed a little wrapper for this library, to use from different environments. We found that [BottlePy](http://bottlepy.org/docs/dev/index.html) was the perfect solution: it's a very simple and light wsgi server; it provided an API-like interface and its only dependence is Python.

This [script](https://github.com/yamila-moreno/jannie-5/blob/master/src/ardrone_server.py) raises a little server which stands in the port 8080. It's in charge of the communication with the libardrone library and provides a cleaner interface. The script is located in the Linux part of the Yún and starts automatically thanks to an [init script](http://www.linux.com/learn/tutorials/442412-managing-linux-daemons-with-init-scripts).

4. Sensors

One key part of this project was managing sensors. And for this task it's really important to know... well, the sensors:

<figure>
  <img src="/images/2014/12/sensors.jpg"
       alt="Some of these, and some more of those..." />
  <figcaption>Some of these, and some more of those...</figcaption>
</figure>

Our initial robotic arm became a glove with some components sewed. We removed leds and weapons (doh!) and focused in the sensors:




* a pressure sensor to takeoff and land the drone
* a flexometer, to activate and deactivate the navigation mode
* an accelerometer, to control the movements of the drone


We made a [simple sketch](https://github.com/yamila-moreno/jannie-5/tree/master/examples/sensortest) to read the values of the sensors and calibrate ranges.

Our first attempt using the sensors...



The second was much better!!



**Protip**: [Using Arduino from command line](http://moduslaborandi.net/arduino-from-command-line/)

And we built the glove:

<figure>
  <img src="/images/2014/12/glove.jpg"
       alt="Así cosía, así así, así cosía así así, (spanish popular song)" />
  <figcaption>Así cosía, así así, así cosía así así, (spanish popular song)</figcaption>
</figure>

5. Arduino

Last piece, but not least was the Arduino sketch and to finish the circuit. We needed to solder some pieces:

<figure>
  <img src="/images/2014/12/solder.jpg"
       alt="Ey, McGyver, I'm like your!!" />
  <figcaption>Ey, McGyver, I'm like your!!</figcaption>
</figure>

<figure>
  <img src="https://raw.githubusercontent.com/yamila-moreno/jannie-5/master/docs/schematic_bb.png"
       alt="The circuit" />
  <figcaption>The circuit</figcaption>
</figure>

In the real world, the circuit was more like that:

<figure>
  <img src="/images/2014/12/circuit.jpg"
       alt="Boards and cables, boards and cables everywhere..." />
  <figcaption>Boards and cables, boards and cables everywhere...</figcaption>
</figure>


Thanks to the [Process](http://arduino.cc/en/Tutorial/Process) library, it's possible to communicate the Arduino with the Linux inside the Yún (Linino, a distribution based upon OpenWRT). The [sketch](https://github.com/yamila-moreno/jannie-5/tree/master/src/ardruino) is really simple and easy to follow.

And that's all!! With all these pieces put together, we could make the drone fly!! At demo time..



And we could even make a bis!



**Post PIWeek**. I have learnt some valuable things:

* fiddling with hardware is so cool! I really love it
* fiddling with hardware sucks because it's always broken
* Linino (the Linux inside the Yún) is a very limited Linux
* the Lilypad is way better, but I don't know if there is a "LilyPadYún", with Linux and WiFi
* the open hardware is an amazing concept and I really hope it developes and evolves, the more the better
* and.. which of you recognize the reference of Jannie-5? ;-)
