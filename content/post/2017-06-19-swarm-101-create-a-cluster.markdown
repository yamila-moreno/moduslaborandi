---
author: yamila
comments: true
published: true
date: 2017-06-19 10:02:22+00:00
slug: swarm-101-create-a-cluster
title: Swarm 101 - Create a cluster
tags:
- docker
- swarm
thumbnailImagePosition: left
thumbnailImage: https://ih0.redbubble.net/image.282529586.5735/flat,200x200,075,f.u2.jpg
---

New tutorial for beginners, this time is an introduction to Swarm, an <a href="https://www.quora.com/What-is-orchestration" target="_new">orchestration</a> tool which comes with Docker. I wrote this tutorial after following the <a href="https://docs.docker.com/engine/swarm/swarm-tutorial/" target="_new">official quickstart</a> and after filtering which I think is the most useful information to start with orchestration.
<!--more-->

Likewise, I'm adding some explanations of what I found most difficult to understand.

<h2>Previously in Kubernetes</h2>

Some months ago, I followed a <a href="https://kubernetes.io/" target="_new">Kubernetes</a> tutorial, the Google orchestrator, which these days have the most traction power. During and after the excercise, I felt like:

<figure>
  <img src="https://i1.wp.com/vinodvihar75.files.wordpress.com/2014/12/dogs-2.jpg?w=290&h=217&crop&ssl=1" alt="Dog typing in a keyboard" />
  <figcaption>Don't know what I'm doing</figcaption>
</figure>

It seemed to me a very sophisticated tool, far from being beginner-friendly. This made sense, because it's a relatively new tool, which affects the maturity of documentation; besides, it was solving a complex problem, outside my knowledge zone. Thus, I concluded that orchestration wasn't my thing.

However, I've spent two weeks studying the basis of Swarm, and now I think there is a way of approaching the question, more suitable for beginners. So, there we go!

<h2>Detailed goal</h2>

I've looked through several Swarm tutorials, and I've seen different approaches, so maybe it's useful to share the specific scope of this tutorial; what I want to show is:

* basic concepts of orchestration: state, replicas, nodes, manager, worker, routing
* basic commands to create a Swarm cluster
* basic commands to deploy docker images in a Swarm
* how to write a <code>docker-compose.yml</code> file in version 3, which can be used in a Swarm

All this information will be split in several posts to make it easier to follow.

<h2>Assumptions</h2>

This tutorial assumes that you understand the basis of docker. If it's the first time you're going to deal with docker, I recommend you to spend a while with this <a href="http://moduslaborandi.net/2016/02/docker-101-hello-world/" target="_new">introduction to docker</a> and with this other <a href="http://moduslaborandi.net/2016/02/docker-101-dockerfile/" target="_new">introduction to Dockerfile</a>.

Besides, I'll use an example that starts after we can put our application in a docker image available through a registry. I published recently an <a href="http://moduslaborandi.net/2017/06/gitlabci-101/" target="_new">introduction to GitlabCI</a> that you may find useful.

<h2>Let's start!</h2>

We'd like to learn how to orchestrate with Swarm. **Orchestration** is the set of tools and practices to deploy applications that can be in HA (high availability, the service is always responding), and where the infrastructure can scale quickly (if there is a peak, the service can be updated seamlessly). To achieve that, we're going to use <em>Swarm mode</em> that comes out-of-the-box with docker. We'll need a **cluster**: the (tipically physical) machines available for creating the nodes for Swarm. Those machines need to have visibility between them.

So we're going to create a cluster in our machine, which replicates a scenario similar to what could be the cloud. With Vagrant, we're using 3 virtual machines (that will be our nodes). You can use this <a href="https://github.com/yamila-moreno/vagrant-cluster" target="_new">Vagrantfile</a> and its provision. If you type <code>vagrant up</code> in the <em>Vagrantfile</em> directory, the system will load 3 virtual machines with ubuntu (and docker).

| Don't try the Swarm commands directly in your machine; when activating the Swarm mode, your docker will be modified and you'll probably need to rollback some changes. The best option is to use the virtual machines in Vagrant.

Swarm works distributing the services among the cluster machines. Each machine, by default is a **worker**; in addition, some machines are **managers**: those managers know the state of the cluster; if a node falls, managers detect it and launch measures to restablish the desired **application state**. In a Swarm cluster there should be always an odd number of managers, so they can use **consensus algorithms**.

So first thing we're gonna do is to initialize one of the machines as a manager (server1) and we're going to associate the rest (server2 and server3) as workers:
```
# enter the server1
(myhost)$ vagrant ssh server1
(server1)$ docker swarm init --advertise-addr 10.0.15.21
```

The output of this command is needed to associate the rest of the nodes to the cluster; the output will be similar to:
```
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

If by any chance, we lose this command (maybe we close the window terminal), we can recover it executing in the manager node the following command:
```
(server1)$ docker swarm join-token worker
```

Now we have to enter in both of the other machines and execute this <code>docker swarm join --token...</code> command in server2 and server3. Done! We've a Swarm cluster up and running. Now, a couple of useful commands:
```
# to see the list of services in a cluster
(server1)$ docker service ls

# to see the nodes in a cluster
(server1)$ docker node ls
ID                            HOSTNAME     STATUS     AVAILABILITY     MANAGER STATUS
niqaah8mhlou9gij2fye046sq *   server1      Ready      Active           Leader
qowv0xmux806zu5u7f7or4yed     server2      Ready      Active
w92gfg7dwl33esmmk7acnhcdk     server3      Ready      Active
```
The asterisk indicates the current node, and <em>Leader</em> tells that this is a manager node.

Well done! So far, we've seen some basic concepts and we've created a cluster. Next post will cover how to create services, update and inspect them. But now, it could be a good moment to stretch your legs. See you in the next post!
