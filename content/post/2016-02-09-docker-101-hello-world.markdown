---
author: yamila
comments: true
date: 2016-02-09 22:31:28+00:00
slug: docker-101-hello-world
title: Docker 101 - Hello, world
wordpress_id: 1047
tags:
- docker
- my modus
thumbnailImagePosition: top
thumbnailImage: /images/2016/02/docker-logo.png
---

![docker-logo](/images/2016/02/docker-logo.png)

This post is a tutorial covering the very basics of Docker. Its goal is to offer a way to start with Docker and containers, and some concepts and tips about how to use it.

Disclaimer: I'm not an expert; on the contrary, I have just started learning, like the past week. Because of this specific situation, I know what is being useful for me to learn Docker. Many of the 101 tutorials are just too difficult (for me!), and this aims to be clear, even if it's not too ambitious.

Let's start!
<!-- more -->



## Assumptions



To follow this tutorial properly, it's necessary that you can work with the linux command line, you know git, and you know a bit python and virtualenv.



## Goals



The goals for this tutorial are:


* you know a bit what Docker is and why it is useful
* you can read a Dockerfile and you can create a simple Dockerfile from scratch
* you have your environment ready to work with docker and docker-compose
* you know what's docker-compose and the docker-compose.yml file
* you know how to deploy your system in a PRE environment
* you have some references to continue on your own

Ua! That's a lot! (There will be different posts for all these topics). So let's talk about Docker to introduce the tutorial.



## What's Docker



[Docker](https://www.docker.com/) is an open source tool to run **containers** with our applications. It relies on the Linux Kernel to work, and the _trick_ is that each containter sees itself as a complete operating system with one isolated process.

Why is this useful? It's no new that isolation and repeatability are good for better software development. In recent years, this idea also applies to deployment issues: avoid downtimes, build more resilient systems, and quick answer to load peaks. In some way, they are like virtual machines, but lighter and much faster.



## Dockerhub



When working with Docker, we use _images_ which contain an operating system and our application. These images usually are a very light operating system, where we install our specific dependencies and our code.

Where do those images came from? Later we'll see this in more depth, but for now, we only need to know that there is a repository of images, both official and non-official: [DockerHub](http://hub.docker.com). So from one of those images, we'll add our needs and we'll create our own version of the image. We can share this new image, if we find it interesting for the community. Spoiler: the images we'll be creating during this tutorial will not be so interesting; sorry folks.



## Installation



Ok, we can start with the interesting part: hands on! You need to install Docker in your system and enable the service. For specific issues concerning installation, visit the official (and quite well written!) [documentation](https://docs.docker.com/engine/installation/).



## Hello, world!



First of all, we're going to get our base image:



    $ docker pull ubuntu:latest




This command automatically search for an 'ubuntu' image with tag 'latest' in the Dockerhub and enables it in our system. We see the list of locally available images:



    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    ubuntu              latest              6cc0fc2a5ee2        3 days ago          187.9 MB




It's time to run the image in a container; typically, running an image means to run **one command** inside the image. This one command can be simple or very complex, as we'll see. For our "hello, world" exercise, we're using a very simple command:



    $ docker run ubuntu:latest echo "Hello, world"
    Hello, world
    $




Waaa! SUCCESS!! You nailed it! Well, maybe it seems a poor victory, because, what did really happen right there? When we run the image with the command **echo "Hello, world"**, docker runs up the image, and when the image is ready, launches our command and, important, finishes.

Next: you can also want to run a container and launch commands manually from inside. For this, our "one command" should be _/bin/bash_, let's see:



    $ docker run ubuntu:latest /bin/bash




Oops! It didn't work, right? Time to introduce some arguments of _docker run_. We can check all the options in:



    $ docker --help




And we can check all the available arguments for our option:



    $ docker run --help




Quite a lot options! These are all the posibilities to run docker; right now, we only need to know a couple of them:



    $ docker run -ti ubuntu:latest /bin/bash
    root@7428a3fdbfbe:/# echo "I'm writing this inside the container"
    I'm writing this inside the container
    root@7428a3fdbfbe:/#




Ok, brief explanation: we run the image with _-ti_ arguments, which means basically _I want to run it interactively_, thus, we run a terminal as the command. Docker runs up the container and now we're inside to do our stuff (in the example, we're just _echoing_ a message). Use **Ctrl+d** to exit the container and return to your terminal.



## The power of the images



In the [DockerHub](http://hub.docker.com) there are a lot of prepared images with our favourite componentes. For example, we can get the [PostgreSQL](https://hub.docker.com/_/postgres/) image, **read its description** and use it out of the box:



    $ docker pull postgres:9.4.5




If you read this documentation, you'll see that by default, this container launches a postgres service, in the (inner) port 5432, with a user named "postgres". Then, let's introduce a couple of new arguments:




  * _-d_: which is the opposite as former _-ti_, meaning _I want to run it as daemon_
  * _--name_: which will assign our custom name to the container instead of the random one

Now, to run a new container:



    $ docker run -d --name postgres_container postgres:9.4.5




No news, good news! If nothing seems to happen, it means that probably everything went ok. We can check it with a new _ps_ command:



    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
    289bdf955634        postgres:9.4.5      "/docker-entrypoint.s"   28 seconds ago      Up 27 seconds               5432/tcp            postgres_container




I have to say, UAU!! This is pretty amazing; step by step, we're making it run. Last part of this introduction is accesing our living container to check something inside. To do this, I'm introducing a new _exec_ command. As we have our postgres container running we can:



    $ docker exec -ti postgres_container /bin/bash




At this point, this command should be mainly clear: whatever "exec" means, we are entering interactively, running a terminal in our postgres_container. And, as the _--help_ says, **exec** "Run a command in a running container".

Once inside the container we can check if postgres if fully available:



    $ docker exec -ti postgres_container /bin/bash
    root@289bdf955634:/# psql -U postgres postgres
    psql (9.4.5)
    Type "help" for help.
    postgres=# \d
    No relations found.
    postgres=#




It worked!!! Cool, huh? So, this beggining part is finished! Congratulations if you lasted until this part; we'll continue with the **Dockerfile** in the [next post](http://moduslaborandi.net/docker-101-dockerfile). I hope to see you there; and if you have any questions about this first post, feel free to use the comments and help me improve the tutorial.

Happy hacking!


