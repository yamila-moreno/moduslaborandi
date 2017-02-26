---
author: yamila
comments: true
date: 2015-01-06 22:28:57+00:00
layout: post
slug: googleoff-i-geocaching-and-cgeo
title: 'GoogleOff (I): Geocaching and cgeo'
wordpress_id: 760
categories:
- Tutorials
tags:
- android
- googleOff
- motivation
- my modus
- open source
---

<a href="http://www.geocaching.com" target="_blank">
  <figure>
    <img src="/images/2015/01/Logo_Geocaching_Horiz_MuddyBootBrown.png" width="500px"
         alt="The Geocaching Logo is a registered trademark of Groundspeak, Inc. Used with permission." />
    <figcaption>The Geocaching Logo is a registered trademark of Groundspeak, Inc. Used with permission.</figcaption>
  </figure>
</a>

Some time ago, I saw I had some important aspects depending on third party applications. Propietary applications in many cases, with some important data. For instance, Flickr (I have lost two accounts in Flickr after the integration with Yahoo), Dropbox, and, of course, Google: mail, calendar, contacts, my cell phone, location...

Those companies owe us. Not to mention that those companies sell us to the first NSA greeting at their door. But, what can we do? To be an hermit without technology? Well, we could do that or... we can be more and more responsible. Please, don't get crazy, this is a **process** of awareness and new responsabilities.

This "GoogleOff" posts are about my process decoupling some important data from the big important companies. Slow process, sorry for the impatients. This first post is about one of my hobbies: [Geocaching](https://www.geocaching.com/play).
<!-- more -->
[Geocaching](https://www.geocaching.com/play) is a world wide game; funny and allows me to find new places. It consist on hidding "caches" (like treasures) and finding the caches that other hide. Recently I found my 300 cache, which is not a big amount but it's MY amount :D

Well, this game became popular with everyone having a smartphone with GPS. The website uses [Open Street Map](https://www.openstreetmap.org/#map=5/51.500/-0.100), but the main mobile application [cgeo](https://play.google.com/store/apps/details?id=cgeo.geocaching) (which is not the official application, although is much better than the official) has Google Maps as default settings. Which means that lots of us use the _out of the box_ configuration. Which also means that you have to accept the terms of use of Google Maps (have you ever read? Scary!!)

If you want to try with Open Street Map, make the following:

1. Create a new folder in your smarthpone for this maps. In my case, I created the directoty `maps` inside `.cgeo` directory.
2. Download here the maps you are going to use. You have to use the maps in this [mapsforge repository](http://download.mapsforge.org/maps/).
3. Go to `cgeo > settings > map` and choose the previous directory in the section _Directory with offline maps_.
4. In the same `cgeo > settings > map`, go to section _Select map source_ and you will see the new maps. Choose the one you are going to need.

An **important advantage** is that you have the maps downloaded in the mobile phone, while Google Maps doesn't allow this option in all the countries. This is specially useful when you are geocaching during a hicking route, without enough signal in the middle of a mountain.

One disadvantage is that you cannot reuse the Open Street Map between applications, so you may have some redundant data.

**Achtung, cuidado, warning**, last version of cgeo cames with a brand new feature: _by default_ (tsk) it uses Google Play services to geolocation. So if you have OSM and you don't allow Google apps to locate you then cgeo doesn't locate you. But there is an easy solution!

1. Go to `cgeo > settings > system`
2. Scroll down until the _Geolocation_ section
3. Uncheck the option _Use Google Play services_

And that's all!! Now you have a bit more independent device. Congratulations! It would be really awesome if you shared your own experiences; I'm sure there are lots of tips to learn
