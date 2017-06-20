---
author: yamila
comments: true
published: false
date: 2017-06-22 10:02:22+00:00
slug: swarm-101-manage-services
title: Swarm 101 - Manage services
tags:
- docker
- swarm
---

Second post of the tutorial <em>Swarm for beginners</em>. This post will cover the creation of services, update them in different ways, inspecting them, publishing ports and removing the services. Now, what's a service?
<!--more-->

A service in a Swarm cluster is each application we want to deploy; tipically, we think of micro-services because those application should tend to be small pieces. Examples of services: API, database, front, backoffice, queues, application server...

You should follow this post after the <a href="http://moduslaborandi.net/2017/06/swarm-101-create-a-cluster/" target="_new">post of how to create a cluster</a>. In this post, we'll deal only with stateless services. If we need a database, we'll put it outside the cluster for now; replicating and synchronizing databases have other concerns that are way far from the scope of this post or tutorial (or my current knowledge :P). The same applies to static files, we can assume that they are out of the cluster (for instance in Amazon S3).

<h2>Creating services</h2>

In the next examples, we're going to use docker images from the <a href="https://hub.docker.com/" target="_new">Docker Hub</a>. First of all, let's create a service. All service management has to be done in a manager node, so we enter the <em>server1</em> virtual machine:

```
(myhost)$ vagrant ssh server1
```

And now we can create the service:
```
(server1)$ docker service create --replicas 1 --name myping alpine ping docker.com
```
Let's go through the command:

- <code>docker service create</code> is quite straigth forward: we want to create a service
- <code>--replicas 1</code> means that we are creating only one replica of the service in the cluster. Later, we'll go deeper into this "replica" option.
- <code>--name myping</code> a unique and meaningful name for the service.
- <code>alpine ping docker.com</code> indicates the images to use in the service (an alpine image) and the command to start the image.

You can see that the options are pretty similar when you run an image with <em>plain</em> docker. Let's see some useful commands; first, list all the services in the cluster:
```
(server1)$ docker service ls
ID              NAME      MODE          REPLICAS    IMAGE            PORTS
bjvi88pqv957    myping    replicated    1/1         alpine:latest
```

The column <em>mode</em> says if the service is deployed <em>replicated</em> or <em>global</em>:

- replicated mode is used to deploy our business services. Swarm will create as many replicas as commanded, and it will distribute them among the cluster. We can find more than one replica in a node of the cluster.
- global mode is used to deploy helpers as monitoring systems, or backups. When Swarm deploys a global service, it creates one and only one replica in every node of the cluster.

Now, let's print the status of one process:
```
(server1)$ docker service ps myping
ID              NAME        IMAGE            NODE       DESIRED STATE    CURRENT STATE             ERROR    PORTS
mi2hqax5l9da    myping.1    alpine:latest    server1    Running          Running 24 minutes ago
```

The number next to the name of the service, <code>myping.1</code>, is like an index of replicas of one service and the <em>node</em> column says in which server it's been deployed.

We can also inspect the service:
```
(server1)$ docker service inspect --pretty myping
ID:		bjvi88pqv957v61ey7sbapnmn
Name:		myping
Service Mode:	Replicated
 Replicas:	1
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		alpine:latest@sha256:0b94d1d1b5eb130dd0253374552445b39470653fb1a1ec2d81490948876e462c
 Args:		ping docker.com
Resources:
Endpoint Mode:	vip
```

Right now, this <em>inspect</em> is more curious than useful, because we're keeping the default settings. If you go further studying Swarm deployments, you'll find this information more meanigful.

Besides, as we are managing docker containers, we can go to the node where the container is deployed and use the docker interface to inspect the state:
```
(server1)$ docker ps -a
```

As well as creating a service, we can remove it:
```
(server1)$ docker service rm myping
```

{{< alert warning >}} You cannot remove a service by removing its containers. If you delete a container, Swarm will be noticed and will create a new replica to keep the state of the service. This is part of the awesomeness of Swarm. {{< /alert >}}

<h2>Updating services</h2>

We can update a service changing the state (number of replicas for instance) or changing the proper service (the docker image, the command, the published ports, etc). Both cases work in the same way, with the command <code>docker service update</code>. Let's go through some examples. First of all, we want to create the same <em>myping</em> service:
```
(server1)$ docker service create --replicas 1 --name myping alpine ping docker.com
```

Now, we're going to update it to add more replicas:
```
(server1)$ docker service update --replicas 3 myping
```

You can see the status of the service:
```
ubuntu@server1:~$ docker service ps myping
ID              NAME        IMAGE            NODE       DESIRED STATE    CURRENT STATE                ERROR    PORTS
mi2hqax5l9da    myping.1    alpine:latest    server1    Running          Running about an hour ago
5y2ifjrrhqh2    myping.2    alpine:latest    server3    Running          Running 3 seconds ago
yt74o8vzg7lt    myping.3    alpine:latest    server2    Running          Running 3 seconds ago
```

Wow! Now we have three replicas, easy peasy! You can see in the <em>name</em> column the indexes I was talking above. Remove this service and let's try a different update. Again, the first step is creating a service:
```
(server1)$ docker service create --replicas 3 --name redis --update-delay 3s redis:3.0.6
```

The only new parameter is <code>--update-delay</code> which indicates the time between updating containers. According to the default settings, when we update a service, Swarm will follow these actions:

- stop one replica
- launch a new replica with the new image
- wait the update-delay time (by default is 10 seconds, and I've made it just faster)
- repeat the three previous steps until there are no old containers left

Open a new terminal and <code>vagrant ssh server1</code>; yes, two terminals in the same virtual machine :D. In one of them, type:
```
(server1)$ watch docker service ps redis
```

In the other terminal, let's update the service:
```
(server1)$ docker service update --image redis:3.0.7 redis
```

Now, take a look at the first terminal; you can see how Swarm is stopping a container and launching one new container; then stopping another container and so on... Congratulations! You've done great! We've only one more topic to cover in this post: routing mesh and publishing ports.

<h2>Ports and routes</h2>

This is probably the part that I found more difficult to understand, so I'll try to make it clear. Until now, we've been creating simple services; usually, we'll need to publish a port, so the external users can reach our service; for instance, an <em>Nginx</em> for serving our application. Once again, let's remove all previous services, and let's create a new one:
```
(server1)$ docker service create --replicas 1 --name nginx --publish 8000:80 nginx
```
With the new <code>--publish</code> parameter we're exposing the port 80 of the containers in the port 8000 of the hosts (in this case, the virtual machines). To check it, open a browser and go to <code>http://10.0.15.21:8000</code> (the IP of the manager node); you'll see the <em>welcome</em> message of Nginx. Then, go to <code>http://10.0.15.22:8000</code> and to <code>http://10.0.15.23:8000</code> (server2 and server3 IPs): you'll see the same message. Notice that we only launched one replica, but you can access the service from the three nodes. This is due to the <a href="https://docs.docker.com/engine/swarm/ingress/" target="_new"><em>routing mesh</em></a>, the system that Swarms uses to manage routes in the cluster.

Every time we create a service that exposes a port, Swarm will open the same port in all the nodes of the cluster, even if those nodes don't have any replicas of the service. In our example, we know that we can access the port 8000 of each node, even though we've only one replica. If a request goes to a node without the Ningx, the routing mesh will redirect the request to a node where an actual nginx is listening.

Mind you, this <em>routing mesh</em> applies only to the exposed ports; inside the cluster, the containers see each other through the overlay network, as it happens with regular docker containers.

You can inspect the nginx service, remove it, create again and update to more replicas. Try to play a bit with the commands so you can feel confortable with the Swarm interface and options. Well done! Thanks for following the tutorial up to this point, and congratulations, you've learned some interesting things about orchestration.

In the next post, you'll learn how to manage a swarm with a <em>docker-compose.yml</em> file with our own project, so it will be even more understandable. And feel free to drop any comments below. Happy hacking!
