---
author: yamila
comments: true
date: 2016-02-10 21:06:20+00:00
slug: docker-101-dockerfile
title: Docker 101 - Dockerfile
wordpress_id: 1053
categories:
- Open source
- Tutorials
tags:
- docker
- my modus
---

This is the second post of the Docker 101 tutorial; you can check [the first one](http://moduslaborandi.net/docker-101-hello-world/) if you missed it. So far we have installed Docker, we've made a couple of different "Hello, world" with Docker and we have learnt basic modes of running (interactive, daemon).

This new post is meant to introduce a new scenario of use (more real, but still simple), and the [Dockerfile](https://docs.docker.com/engine/reference/builder/); so we'll learn how to build custom images.

Let's go!
<!-- more -->



## Our new scenario



In this part of the tutorial we're going to work with a real project, in its simplest form, to go through the main interaction. I've prepared a very simple project so you can use it; now you'll need both _git_ and _python_ knowledge. If you don't, you can ask for some help or you can be imaginative creating a similar environment.

The code is available [in my Github](https://github.com/yamila-moreno/api-example). It's a very simple API made with [Anillo]() and Python. To get ready with your environment:




  * You need to copy _myapp.ini_ and _pytest.ini_ files outside the repository
  * You need to export two env vars: MY_APP and MY_APP_TEST, pointing the previous files
  * You need sqlite (usually it's in all Linux)
  * I encourage you to use a virtualenv (with Python3!!!)
  * You need to install _requirements.txt_ and _requirements-server.txt_



Once you've all of this, you can run the API locally directly with Anillo:



    (myapp) src/ $ python run.py serve --create-db --with-fixtures --no-hot-reload




Or, if you installed _requirements-server.txt_, you can run the application with Gunicorn:



    (myapp) src/ $ gunicorn -b 0.0.0.0:5005 --access-logfile - --error-logfile - --log-level debug 'wsgi:load_application("create-db", "with-fixtures")'




You can check it visiting _http://localhost:5005/api/v1_ or _http://localhost:5005/api/v1/users_. Yay! You have a running API in your computer! ;-) Congratulations!

Well, this was not about docker, was it? Be patient, this was the project we're going to _dockerize_, so it's interesting spending some time to have the same environment. Now, why would we use docker for this? For instance, we may want to run automatically the tests, and if they pass, deploy our API in PRE environment connected to a postgres container. To do this, we need an image with our API (we already cloned it in our machine manually), with all the dependences (we installed them manually) and with the proper command (which we launched manually). How can we do an automatic process with so many manual operations?

[Dockerfile](https://docs.docker.com/engine/reference/builder/) will help us!



## Dockerfile



[Dockerfile](https://docs.docker.com/engine/reference/builder/) is a file which sets a "FROM" image, and adds all "the commands to assemble an image". The simplest Dockerfile could be:



    FROM ubuntu:14:04




Just a file called _Dockerfile_ with this single line is neccessary to build our own image. Let's see how to build an image:



    $ docker build -t my-image:1.0 .




You have to run it in the directory where the Dockerfile is. The _-t_ option sets the name and tag for the new image. Now, you can check you have a new image available:



    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    ubuntu              latest              6cc0fc2a5ee2        3 days ago          187.9 MB
    my-image            1.0                 6cc0fc2a5ee3        11 days ago         187.9 MB




Take some minutes, go over the [previous post](http://moduslaborandi.net/docker-101-hello-world/) and try to run the same "Hello, world" exercises with the new images. Great!!!



## Our Dockerfiles



Let's create a couple of Dockerfiles to build our images. Those images will be:




  * An image with postgres and a database called "myapp"
  * An image with the API, conected to the database, serving the API with gunicorn



To do this, we need this tree directory:



    docker-101
    |__api-example
       |__src
       |__docker
          |__postgres
          |__api




You can find all these files in the repository, but I invite you to write them on your own. All commands from now will be executed from _docker-101_ directory.

Next: create a _Dockerfile_ in _postgres_ directory, with the following content:



    FROM postgres:9.4.5
    MAINTAINER Yamila Moreno yamila.ms@gmail.com
    ENV POSTGRES_DB myapp




What's the meaning of this Dockerfile?




  * The FROM statement says that we are starting from a postgres (official) image, in its version 9.4.5
  * The MAINTAINER line just says who am I, and who should you send the huge amounts of money ;-)
  * As the [documentation](https://hub.docker.com/_/postgres/) says, if we build the image with "POSTGRES_DB" env var, the value will be used as the name of the database. We use the statement ENV.



Easy peasy. Let's build our image:



    $ docker build -t myapp-postgres:1.0 -f api-example/docker/postgres/Dockerfile .




And check the new images exists. Let's run it to check if everything went ok:



    $ docker run -d --name myapp-postgres myapp-postgres:1.0




Now, enter in the container and execute psql:



    $ docker exec -ti myapp-postgres /bin/bash
    root@ec861b83cdee:/# psql -U postgres myapp
    psql (9.4.5)
    Type "help" for help.
    myapp=#




See that we are entering in "myapp" database, instead of "postgres" default database. So yes, everything went well!

Next, let's do the same operations for our API, which is a bit more complex. Create a _Dockerfile_ in _api_ directory with the following content:



    FROM ubuntu:14.04
    MAINTAINER Yamila Moreno yamila.ms@gmail.com
    WORKDIR /

    # Install dependencies
    RUN apt-get update
    RUN apt-get install -y -qq curl python3-pip git libpq-dev
    RUN pip3 install virtualenv

    # Configure locales
    RUN locale-gen "en_US.UTF-8"
    RUN dpkg-reconfigure locales
    ENV LC_ALL "en_US.UTF-8"

    EXPOSE 5005

    # Setup the application
    RUN virtualenv -p python3 venv
    COPY api-example /api-example
    RUN /venv/bin/pip install -r /api-example/requirements.txt
    RUN /venv/bin/pip install -r /api-example/requirements-server.txt

    # Set specific application env vars. We're using the default configuration
    ENV MY_APP=/api-example/src/settings/myapp.ini

    # Run application
    WORKDIR /api-example/src
    CMD /venv/bin/gunicorn -b 0.0.0.0:5005 --access-logfile - --error-logfile - --log-level debug 'wsgi:load_application("create-db", "with-fixtures")'




Lots of things!! Let's go through each new line:




  * EXPOSE: informs the environment is exposing this port. It's a good practice
  * WORKDIR: just indicates that next command will be executed from that directory
  * RUN: runs a command. In our case, we're updating the system and installing some dependencies. We also create a virtualenv.
  * CMD: remember the "one command" we have talked before? This is THE one command! In fact it's an array of commands, but in our example, we just need one.

Now, we can bulid the new image:



    $ docker build -t myapp-api:1.0 -f api-example/docker/api/Dockerfile .




And again, run it:



    $ docker run -d --name myapp-api myapp-api:1.0




Now, we should find our API in _http://localhost:5005_; but if you have followed my instructions, you won't find anything. Why? In fact, lets run a different image and make the same test:




    $ docker run -ti --name myubuntu ubuntu:14:04 /bin/bash
    root@ec861b83cdee:/# apt-get install curl # yes, you need to install it
    root@ec861b83cdee:/# curl http://localhost:5005




Still not working. This is because, by default, docker doesn't share anything, nor with the host (your machine), neither with the other containers.

**Visibility between containers**

First of all, we want to make our containers visible between them. Time to introduce new concepts:

_--link myapp-api:myapp-api_; it's an option when running the image that creates a link between the container you are running and the container in the parameter. Here we're saying that the container running our image is linked to "myapp-api" with the name "myapp-api".

So, let's stop the ubuntu machine:



    $ docker stop myubuntu




And run it again linking the containers:



    $ docker run -ti --name myubuntu --link myapp-api:myapp-api ubuntu:14.04 /bin/bash
    root@ec861b83cdee:/# apt-get install curl # yes, you need to install it. Yes, again. Each time.
    root@ec861b83cdee:/# curl http://myapp-api:5005/api/v1 # we are using the mapped name
    {"api-version": "1.0", "message": "Welcome to this API."}




At last!! It worked!! :__) It's being a bit hard, isn't it? but we're learning step by step lots of concepts. Take a moment to read again any part and make different tests. And take a moment to watch over your screen, through the window, where the sun shines and the wind... winds... ;-) And let's move on!

Note about "link": this argument is to be deprecated and soon; the "network" feature will cover the old functionality and will add more. When I'm writing this post, "network" is still not released in stable version, so I'm using links.

**Visibility with the host**

We have seen how to make the containers visible between them; but usually, we'll need that some containers are visible from the host. And time to introduce new concepts:

_-p 5433:5432_; it's an option when running the images that exposes the container's port (5432) in the host's port (5433). I'm mapping in my 5433 because my own 5432 is "busy" with my own postgresql.

Let's test it with our postgres image:



    $ docker run -d -p 5433:5432 --name myapp-postgres postgres:9.4.5




And now, access the database from our machine (you'll need to have _postgres_ installed):



    $ psql -h localhost -p 5433 -U postgres postgres
    # And you'll see some output like:
    Line style is unicode.
    Border style is 2.
    Output format is wrapped.
    psql (9.4.5)
    Type "help" for help.

    postgres=#




There you are! This database is running in a container :D But previously we were trying to make the api accesible from the host. Take a moment to think how would you do this and then continue reading.

Stop and delete running containers and run the api from scratch with the new param:



    $ docker run -d -p 5005:5005 --name myapp-api myapp:1.0




And then, from our machine:



    $ curl http://localhost:5005/api/v1
    {"message": "Welcome to this API.", "api-version": "1.0"}%




Total success!!

This post has been a bit dense, with many new concepts, not easy at all (dockerfile, link, port mapping). Congratulations to get here. The reward will be the next post, where we will make it easier to deal with all these running options. We'll talk about _docker-compose_.

Happy hacking!





