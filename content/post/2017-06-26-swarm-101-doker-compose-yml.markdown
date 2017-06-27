---
author: yamila
comments: true
date: 2017-06-27 10:02:22+00:00
slug: swarm-101-docker-compose-yml
title: Swarm 101 - docker-compose.yml
tags:
- docker
- swarm
---

Third and last post of the tutorial <em>Swarm for beginners</em>. After <a href="" target="_new">creating a Swarm cluster</a> and learning the <a href="" target="_new">the basis of services management</a>, it's time for deploying our services in the cluster using <em>docker-compose.yml</em> as we use it with docker.
<!--more-->

You can use <a href="" target="_new">this simple project</a> to follow the tutorial. All the images and versions in this post point to <em>mymac</em> project. To deploy with docker-compose.yml,

<h2>docker-compose.yml</h2>

First of all, we create a file called <code>docker-compose.yml</code>; it will have the same structure as <a href="http://moduslaborandi.net/2016/02/docker-101-docker-compose/" target="_new">the well known docker-compose.yml</a>, and it will work seamlessly with docker swarm.

Create a file called <em>docker-compose.yml</em> in <code>server1</code>. If you are using the <em>simple mymac</em> project, you can copy and paste directly. Otherwise, change whatever you need to start your project. The file should be something like:
```
version: "3"

services:
  mymac:
    image: registry.gitlab.com/yamila/python-back:22528ebd5efb936249bd7bd8f3b6cf6b3113bdc3
    ports:
       - 8000:8000
    command: gunicorn --bind 0.0.0.0:8000 wsgi:app
    deploy:
       replicas: 8
```

In version "3" we can add the <em>deploy</em> section where we specify the options for creating or updating a docker service. In the above example, for instance, we have declared 8 replicas (the same as <code>--replicas 8</code>). Let's create this service:
```
(server1)$ docker stack deploy --compose-file docker-compose.yml mystack
(server1)$ docker service ps mystack
```

- <code>docker stack deploy</code> is the main command
- <code>--compose-file</code> is the param which points to the <em>docker-compose.yml</em> file.
- <code>mystack</code> which is the name for the stack (a bundle of services).

Now, we can test this. Go to http://10.0.15.21:8000 in a browser and press "F5" several times; you'll see how the message changes with different macs and ips. Cool!

<h2>Updating services</h2>

If we need to update the image for our service in the stack, we can just change the image reference to this one:
```
image: registry.gitlab.com/yamila/python-back:53d64b4f1b12ede003531b7aecb61c5fcf2613b7
```
Time to update:
```
(server1)$ docker stack deploy --compose-file docker-compose.yml mystack
(server1)$ watch docker service ps mystack_mymac
```
Same as before, this command will show you how the system is stopping a container and launching a new one with the new version. While the system is being updated, you can check in the browser, pressing "F5": sometimes, you'll have the old backend ("Hello, my hostname is xxxx and my ip is zzzz") and sometimes, you'll see the new backend ("Hello, my hostname is xxxx"). Eventually, all the requests will return the new backend, which means that the system is completely updated.

Congratulations! You get to the end of the tutorial. Following the three posts, you've learnt the basis of creating a cluster of Swarm, managing the services manually, and how to manage services and stacks with <em>docker-compose.yml</em> file. Besides, you've learnt lots of concepts that will apply the same to other orchestrators. Grab a cup of tea or go for a walk, you deserve it!!!

If you liked the tutorial, or you have any questions, don't hesitate and use the comments sections. I hope it's been both useful and interesting. Happy hacking!
