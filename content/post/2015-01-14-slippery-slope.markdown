---
author: yamila
comments: true
date: 2015-01-14 00:03:43+00:00
layout: post
slug: slippery-slope
title: Slippery Slope
wordpress_id: 768
categories:
- General
---

I came from winter holidays and wanted to share some photos in my [travel blog](http://dendarii.es). That triggered a chain reaction with a series of unfortunate events...
TL;DR: Check [this repo](https://github.com/yamila-moreno/py500px); simple usage of [500px](https://500px.com).
<!-- more -->

So I wanted to add a blog post to share my winter trip along Aquitanie. First thing was uploading my photos to Flickr. Flickr had added a two steps login, which consisted on sending a code to my mobile phone. But I had given a wrong mobile phone BECAUSE YES**. After some emails, I finally could recover my account, but, by the moment, I had decided to migrate to a new photostream platform.

I found **500px**. It looks nice, and it's not so massive. And it wasn't mandatory to set the gender (WTF [Ipernity](http://www.ipernity.com/) and your "_technical reasons_"). So I created my new account.

I manually uploaded my first set of photos, and looked for some widget to generate embeded galleries for my blog. I found one which only worked with tags; but my new sets didn't have, and... the web doesn't allow you bulk edition. [Three years ago](https://twitter.com/500px_Help/status/271942761559760896) they said bulk edition was coming. It's still coming.

But, it didn't matter! I found they have a public API, with documentation!! And, am I a developer or not? And specially, am I stupid enough to yield when I see a public API? OF COURSE!! And I started [this little project](https://github.com/yamila-moreno/py500px) to make some little bulk editions.

I also found (so many discoveries) that 500px offered, thanks to Flickrpicker.io (sort of), migration from Flickr. At last! Good news! Nope... The migration tool is simply useless. But they recommended me to use their API.

So when I found myself, struggling in the nigth against three APIs, and the combination of the three of them didn't allow the really simple things that I needed, I reconsidered my way of living and I'm probably going to sail the sea, and go to places... without internet signal ;-)

As a result, there you have a very simple and very improvable Python project to give names and tags to collections. I hope you enjoy the project, or at least this slippery slope story.

ps. The blog post, of course, is not yet published. Or even written.
** my story with Flickr and Yahoo cames from long ago. I had _another_ account. I created it with fake data BECAUSE YES. Fake data, like birthdate. I didn't remember what date I gave. But it was very young... Because one day, the system told me I was too young to log in. After serveral attempts to recover my account, I left it there and created a new Flickr account... Shouldn't I love technology, I would hate it ;-)
