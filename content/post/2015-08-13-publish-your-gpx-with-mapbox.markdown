---
author: yamila
comments: true
date: 2015-08-13 18:05:32+00:00
slug: publish-your-gpx-with-mapbox
title: Publish your GPX with Mapbox
wordpress_id: 925
categories:
- Open source
- Tutorials
tags:
- mapbox
- my modus
- open source
---

Following my learning about [maps as a service](http://moduslaborandi.net/publish-your-gpx-with-cartodb/), I've tried also [Mapbox](https://www.mapbox.com/) with the same very simple approach: share a GPX shown in a map, an in a nice way.

After the little tutorial I'll write some impressions comparing both systems.

<!-- more -->



## Assumptions



You need an account in Mapbox. You know how to track your own routes and know what's a .gpx file.



## Create the map



1. Download the .gpx in your computer
2. In Mapbox, go to "Projects" and click in "+ New project" (blue button). This action will take you to a map with some actions to perform.
3. First of all: choose "Style" tab, where you can select from different layouts. For my .gpx I thought that "Outdoors" fit well.
4. Now, "Data" tab. There, you can "import" (little link) a .gpx. Select your .gpx file from your computer and import it. Then "Finish importing"
5. Congratulations! You now see your .gpx in the map.



## Customize the map



6. Now the route is an element of the map. You can double click above, and set name, description and even the colour of the line. You also can add a marker for the beginning or the end of the route, to make it nicer. Or maybe a marker to sign a beauty spot, and link to an external page.

You can also make some other customizations (remember to "Save" from time to time, to keep your changes!):




  * In _Project > Settings_ you can set a name and a description of the map
  * In _Project > Settings_ you should check "Save current map position", so when displayed, always starts with this zoom level in those coordinates





## Share and embed



7. Now it's time to share and embed. To share, go to _Project > Info_ where you'll find a simple url to share. For instance, [this url](https://a.tiles.mapbox.com/v4/yamila.n4i41me8/page.html?access_token=pk.eyJ1IjoieWFtaWxhIiwiYSI6IjUzNDE5ZDRkZjBiZjBiZDY0YTBhZjBmNmUyZGYzYTZiIn0.okLJEzGsBQ6IOgn1mhToIQ#16/51.1375/1.3558).
8. Embeding is easy although you need to be careful about the position and zoom level. Go to _Project > Info_. There you have some checkboxes to customize the embeding. Select what you want and copy the "Embed" input text. Something like this:

```
<iframe width='100%' height='500px' frameBorder='0'
        src='https://a.tiles.mapbox.com/v4/yamila.n4i41me8/attribution,zoompan,zoomwheel,geocoder,share.html?access_token=pk.your_token'>
```

And the trick! Pick the last part of the url (point 7): **"#16/51.1375/1.3558"** where you can see: "#zoom_level/lat/lon". Don't touch lat and lon, but you may want a different initial zoom when embeding. Finally, you'll end with a code like this:

```
<iframe width='100%' height='500px' frameBorder='0'
        src='https://a.tiles.mapbox.com/v4/yamila.n4i41me8/attribution,zoompan,zoomwheel,geocoder,share.html?access_token=pk.your_token#16/51.1375/1.3558'>
```



## The pitfalls



To see the changes reflected (in the embeded mode) [the official documentation](https://www.mapbox.com/help/set-map-position/) says it can take up to 45 minutes. This slow feedback is frustrating because you need to wait to know if you did something wrong. Luckily, you haven't done anything wrong :P

And that's mostly all! You can see the map embeded here:



For this little recipe, I used the [official documentation](https://www.mapbox.com/guides/), where you can find more tutorials and howtos.



## The comparison



With my very little experience with Mapbox and CartoDB (both _as a service_):





  * Mapbox has an easier setup
  * but the customization of the iframe is unnecesaryly difficult
  * CartoDB has more options from the beginning (too much for me, but you may like it)
  * Mapbox didn't answer a tuit that I wrote asking for help
  * CartoDB has more options of customization / animations
  * The 45 minutes-feedback is frustrating
  * Both have pretty user interfaces!



For now, I'll keep with Mapbox because it's simplier, and because I've read very good things about them as a company. Nevertheless, both options are very good so, choose whatever fits best with your usage.

Enjoy!


