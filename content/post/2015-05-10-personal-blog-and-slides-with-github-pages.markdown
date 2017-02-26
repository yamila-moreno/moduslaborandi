---
author: yamila
comments: true
date: 2015-05-10 17:40:55+00:00
layout: post
slug: personal-blog-and-slides-with-github-pages
title: Personal blog and slides with Github Pages
wordpress_id: 825
categories:
- General
---

You may have seen personal pages, blogs or even documentation in [github.io](https://pages.github.com/) domain. Here you have a simple howto about this service of [GitHub](http://github.com).

![github.pages](/images/2015/05/github.pages.jpg)

<!-- more -->



## Assumptions


You have an account in GitHub and you know the very basic of git.



## Scenarios


This post is covering the following scenarios:
- you can use it for your personal page / blog
- you can use it to share documentation / slides
... but you can see how it can be easily applied to other contexts. Up to your needs.



## Personal page


My colleague [@dialelo](http://twitter.com/dialelo) for example has his blog in [http://dialelo.github.io](http://dialelo.github.io). For this, you need a repository called "your-github-user.github.io". I made an example for myself, you can check it in my [github account](http://github.com/yamila-moreno/yamila-moreno.github.io).

You need to add a file _index.html_ in the branch **master**. From this moment, you will have this _index.html_ served (always the branch **master**), in http://your-github-user.gitub.io. You can check my own example in [http://yamila-moreno.github.io](http://yamila-moreno.github.io).

A typical use for this scenario could be:
- a branch **raw** with the source content (markdown for instance)
- a branch **master** where you can generate static html from the source (Jekyll or Pelican). This branch is automatically served.



## Publishing slides with revealjs


First of all, you have to create a new repository for your talk. I have created [this one](https://github.com/yamila-moreno/learnt-lessons-in-a-django-project). In **branch master**, you can put the link to the slides in your README.md. The link should be something like:

_http://your-github-user.github.io/the-repository-of-your-slides_

Then, you have to create a new branch called **gh-pages** (the name matters!!). In this branch, add a file called _index.html_ which will be served from this moment (always branch **gh-pages**) in http://your-github-user.github.io/the-repository-of-your-slides.

But, we want to use revealjs. [RevealJS](http://revealjs.com) is a javascript library to create slides. It supports HTML and markdown, and has lots of features. For the scope of this post, we are keeping in the simplest features.

In this **gh-pages** branch, download all the content of revealjs, from the [official repository](https://github.com/hakimel/reveal.js/releases). You can see that revealjs includes an _index.html_ file, so you just replace all the previous files with the ones you just downloaded. Now, you have the example slides of revealjs; edit the _index.html_ file with your own content, and you will see it online after pushing to Github.

For instance, I created this [repository](https://github.com/yamila-moreno/learnt-lessons-in-a-django-project); then I added a branch **gh-pages** and the revealjs source. So  now you can check this in [http://yamila-moreno.github.io/learnt-lessons-in-a-django-project/#/](http://yamila-moreno.github.io/learnt-lessons-in-a-django-project/#/).

To sum up, the typical use for this scenario would be:
- you have your branch **master**, with LICENSE and README.md (as it's the default display in github)
- you put the revealjs slides in the branch **gh-pages**.

From this point, you can explore the features of revealjs, or the use of [Pelican](http://docs.getpelican.com/en/3.5.0/) to create your blog. Enjoy!
