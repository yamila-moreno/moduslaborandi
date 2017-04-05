---
author: yamila
comments: true
date: 2017-03-25 10:02:22+00:00
slug: sourced-tech-talks
title: Sourced Tech Talks
categories:
- General
tags:
- infrastructure
- event
- talent
---

Yesterday, it was the first edition of <a href="http://talks.sourced.tech" target="_new">Tech talks</a> organized by <a href="http://sourced.tech" target="_new">Source{d}</a>. They want to organize one of those every three months focusing on different aspects of technology. This first edition was about infrastructure.
<!--more-->

<figure>
<img src="https://c1.staticflickr.com/3/2871/32812484614_9957b3b9e3.jpg" />
<figcaption>Source{d} Logo</figcaption>
</figure>

It was a little conference (about 70 people, we were only 6 women counting with the organizers, tsk) very well organized. The offices of source{d} are very open and amazingly decorated. They provided food and beverages for all the day (I would suggest some healthier food, like fruit or salad) so we could just enjoy the conference. For sure, the strength of the conference were the advanced topics that the speakers covered:

* <em>Building a Billion User Load Balancer</em>: <a href="https://twitter.com/servsockd" target="_new">James Reilly</a> is a Network Production Engineer at Facebook. His talk showed how the load balancer system works in Facebook, using different load balancers at different OSI levels. His mission is to create a network which responds as quick as possible to all the billions of requests (figures at FB are pretty awsome): for this, they have a "cartographer", a tool that tests similar requests with different network routes and monitors the result; that way they can find the shortest path between a request and its corresponding server. A very interesting (and accesible!) talk.

* <em>One API to rule them all, Rise of the Operators</em>: <a href="https://twitter.com/ipedrazas" target="_new">Iván Pedrazas</a> is a Kubernetes specialist. In his talk, he tried to show how to use Operators, and advanced feature of Kubernetes, and custom controllers to get your infrastructure to the next step of automation.

<figure>
  <img src="http://talks.sourced.tech/img/venue/lovelace.jpg" />
  <figcaption>Ada Lovelace, painted in their wall</figcaption>
</figure>

* <em>High-performance Linux monitoring with eBPF</em>: <a href="https://twitter.com/2opremio" target="_new">Alfonso Acosta</a> is a Software Engineer at Weaveworks. His talk was a comprehensive explanation about Extended Berkeley Packet Filter (eBPF) a tool that allows introspection to the Linux Kernel execution; it provides high-performance tracking packages at system level, so it could serve for monitoring purposes. This was also a very good talk.

* <em>From 0 to anomaly detection in your infrastructure metrics in 15 minutes</em>: <a href="https://twitter.com/lekum" target="_new">Alejandro Guirao</a> is a Software Engineer at Intelygenz. His lightning talk presented a proof of concept about how HTM (Hierarchical Temporal Memory) can be useful to detect anomalies in the system.

* <em>Booting up your cattle with Ignition and Matchbox</em>: <a href="https://twitter.com/tekess" target="_new">Sonia Meruelo</a> is an Infrastructure Engineer at Source{d}. Her lightning talk presented two important tools, Ingnition and Matchbox, to easily create and provision bare metal servers.


* <em>The many journeys of a disk write</em>: <a href="https://twitter.com/deibyz" target="_new">David Fernández</a> is SRE at MongoDB. He gave us a very unexpected (for me) talk about storage disks, pros and cons of the different writting systems. I'd never had thought about a garbage collector at hardware level (I'm looking at you, SSD).

* <em>Native, Distributed Storage For Kubernetes</em>: <a href="https://twitter.com/asynchio" target="_new">Joshep Jacks</a> is an EIR at Quantum Corporation. He presented Rook, <em>"Open source file, block and object storage for your cloud-native environment"</em>. The main point of the talk was the integration of Rook and Kubernetes.

5 talks and 2 lightning talks. All of them totally worthy, even if I couldn't follow some talks. As a "pure" backend developer, it was surprising to learn about the worries of the infrastructure teams. I feel in the bleeding edge with my `docker-compose.yml` and those speakers didn't even mentioned Swarm. So I just fell down to the <a href="https://en.wikipedia.org/wiki/Dunning%E2%80%93Kruger_effect" target="_new">valley of despair</a> without doing the summit of the <a href="https://understandinginnovation.files.wordpress.com/2015/06/dunning-kruger-0011.jpg" target="_new">Peak of Mt. Stupid</a>, which seems unfair ;-)

I really enjoyed the talks, I had the opportunity of good conversations in the breaks and I can say I learnt a bunch of things \o/
