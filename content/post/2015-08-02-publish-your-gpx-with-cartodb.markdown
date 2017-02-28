---
author: yamila
comments: true
date: 2015-08-02 20:51:38+00:00
slug: publish-your-gpx-with-cartodb
title: Publish your GPX with CartoDB
wordpress_id: 922
categories:
- Open source
- Tutorials
tags:
- cartodb
- my modus
- open source
---

I like go hiking and I usually track the route. Not so long ago, the only way to share and embed this tracks (.gpx) was to convert them to .kml format and publish in Google Maps. Today I've tried to publish my [last route](http://dendarii.es/windmills-path-desde-hassocks-hasta-lewes/) with [CartoDB](https://cartodb.com/) and I'm very happy with the result.

This is a simple recipe to share a .gpx route and embed in your own website.

<!-- more -->



## Assumptions



You need an account in CartoDB. You know how to track your own routes and know what's a .gpx file.



## Create the map



1. Download the .gpx in your computer
2. In CartoDB, click in "New map" (green button), and choose again "Create new map" from scratch
3. In the next step, go to the section "Connect DataSet" > "Data file". Click on "Select a file" (blue button) and select your .gpx
4. Then, click on "Connect a dataset" (green button) and wait for it.
5. If everything went ok, you'll see your route on a map. Congratulations!!



## Customize the map

6. If you don't like how the route is displayed on the map, you can change different settings:
  * in the bottom left of the window, you'll be able to change the basemap and some options
  * in the right side of the window, you'll find some options to change colour and form of the route





## Publish and embed



7. Once you're happy with the map, click in the "Publish" button, copy the "Embed it" code and paste into your webpage / blog post.

You can see the [map](https://yami.cartodb.com/viz/8ab61126-3953-11e5-a325-0e018d66dc29/public_map) in CartoDB and also how it's embeded:



For this little recipe, I used the [official documentation](http://docs.cartodb.com/tutorials/adding_geometries.html), where you can find more tutorials and howtos.

Enjoy!
