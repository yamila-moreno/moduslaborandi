---
author: yamila
comments: true
date: 2015-06-02 14:27:49+00:00
layout: post
slug: migrating-from-wordpress-to-ghost-the-hard-way
title: Migrating from Wordpress to Ghost (the hard way)
wordpress_id: 863
categories:
- Open source
- Tutorials
tags:
- ghost
- wordpress
---

Previously in **Modus laborandi** we learnt how to install and run a Ghost blog. This new post is about migrating the data manually, from Wordpress to Ghost.

<!-- more -->



## Assumptions



You need to know very little about how to read a json file, some skills with find&replace; (vim macros will be useful), and patience :D

I'm assuming that you have the wordpress in your own server, so you can access your images directly.



## Get the data



To get the data, you have to install the [ghost plugin](https://wordpress.org/plugins/ghost/) for wordpress in your blog. Following the instructions for exporting data (not images), we'll end with a json file containing all the data to migrate.



## Test environment



As the process is quite laborious (lots of proof and error), I recommend to run ghost locally and load as many times as needed until we have our data as we want. So, take a minute to [this post](http://moduslaborandi.net/deploy-a-ghost-blog/) and follow it until the **npm start --production** part. With that, you'll have the blog running in your local machine.

Once you have your ghost running, if you want to import the data (which is the json file), you need to:


* go to **http://localhost:2368/ghost/debug**
* **Delete** previous data
* Select the json file in the **import** menu
* Press **Import** blue button

If you haven't touched anything, this import should fail for any reason; don't worry. Even if it goes well, we'll change some things in the json. So, whenever I mention in the post to **reimport**, what I mean is to do all the previous steps (deletion included, yes!).

**WARNING, ACHTUNG, OJOCUIDADO** Deleting all data, deletes all data. And no data remains. Even if this is obvious, I have failed in deleting a database enough times to know that a new warning won't hurt. So please, be sure you are woking in a test environment.



## Drafts



The first error I found importing data was because the ghost plugin doesn't fill the field "published_at" when the post is "private" or "draft". The json for those posts is something like:




    ...
    "posts": [
        {"id":1, ..., "status": "draft", "published_at": ""}
    ]




The system will fail if this field is empty. So the first thing that you need to do search in the json for the posts in "draft" status, and fill the "published_at" field. I used the value in the "created_at" field for each post. From that point, you'll find that the import system doesn't complain about dates format.



## Div tags



Even though the markdown of ghost admits html tags, for me, the **div** tags were useless in my blog, so I removed it:

- remove **<div ...>**
- remove **</div>**



## Images



This part is the one that took me more time. You need to copy or move the pictures from the former **your-wp-blog/wp-content/uploads** to **your-ghost-blog/content/images** (see that you keep the file tree). And then, it's time for mayor changes in the json file:

 * Find urls for images, something like **http://dendarii.es/wp-content/uploads/2015/03/image.png**, and change this into somethinkg like **/content/images/2015/03/image.png**
 * Find and (possibly) delete smilies, which are in urls like **http://dendarii.es/wp-includes/images/smilies/icon_XXX.gif**
 * Be sure that all the images have a linebreak before and after (to be rendered properly)

With this, try to reimport data and take a look. It should be nicer, with all the images in the proper url.



## Bonus 1: markdown tags in images



With the migration, most of the images will look like:




    [![stonehenge2](/content/images/2015\/04\/stonehenge2.jpg)](/content/images/2015\/04\/stonehenge2.jpg)




In ghost, we have some tags to show this pictures prettier. What I did was convert the previous urls into this:




    [![stonehenge2](/content/images/2015\/04\/stonehenge2.jpg#small)](/content/images/2015\/04\/stonehenge2.jpg#full)






## Bonus2: header image



Depending on the theme for ghost, you may want a header image. For that I went through all the posts in the json file, and for each post add a field "image" with the value of any of the images of the post.



## My blog!



And finally, this is how I published [my travel blog](http://dendarii.es)!!

**Note**: for almost all the transformations in json I've used vim macros. They are powerful, and they are difficult. So be patient looking for the proper expression to transform data.

Those are the steps that I needed for my blog, but maybe you'll need another ones to make the migration properly; be creative and don't hesitate to ask whatever you need ;-)

