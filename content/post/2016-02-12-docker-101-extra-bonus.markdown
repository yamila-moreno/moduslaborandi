---
author: yamila
comments: true
date: 2016-02-12 13:29:39+00:00
slug: docker-101-extra-bonus
title: Docker 101 - Extra bonus
wordpress_id: 1072
tags:
- docker
- my modus
---

Well, sometimes life is wonderful and sometimes sucks. Sometimes all of this happen simultaneously. While I was writing the awesome previous posts (mind you, AWESOME), Docker released a new stable version with some important changes. So I found myself deprecated. In a single week.

You were noticed about some of the new stuff, the new **networking** system, but there are more changes, like **volumes** and a "version2" of **docker-compose.yml** to deal with all the changes. Enough for an entire post. So let's go through the changes.

<!-- more -->

So far, we had images running in containers. Now we have also **networks** and **volumes** as "first class citizens". This means that we can create networks as well as volumes.



## First of all



Make sure you have the latest version or so of docker (1.10) and docker-compose (1.6.0). And let's rock! (maybe, just let's try to rock, we'll see).



## Volumes



Volumes are directories that you want to live longer, beyond a container. Until now, the typical way to create a volume was to create a container exposing a VOLUME in the Dockerfile, or in the _docker run_ with the _-v_ argument. Anyhow, whenever the container were removed, the volume was also lost. So: volumes were dependent on containers.

With the new version, they are independent, and we can:



    # create a volume
    $ docker volume create --name my_volume

    # list available volumes
    $ docker volume ls
    DRIVER              VOLUME NAME
    local               my_volume

    # inspect a volume
    $ docker volume inspect my_volume
    [
        {
            "Name": "my_volume",
            "Driver": "local",
            "Mountpoint": "/var/lib/docker/volumes/my_volume/_data"
        }
    ]

    # and delete a volume
    $ docker volume rm my_volume




A couple of notes concerning this:




  * New style volumes are not created with a Dockerfile (but you can still create the old fashioned VOLUMEs); only in command line or in the docker-compose.yml (see below)
  * The mountpoint is fixed (AFAIK), so you cannot map it to a directory in your host



Once you've created a volume, you can attach it to a container as usual:



    $ docker run -ti -v my_volume:/opt ubuntu:14.04 /bin/bash




Exercise: create a volume, and attach to a container. Inside the container, create a file and confirm the file is created in your host machine; then, stop and delete the container and confirm the file is still there, in your host machine.



## Networking



This new feature was presented some months ago, and now it's released. Until now, there were different solutions for networking between containers which worked over Docker: Calico or Weave for example.

**Little explanation required**: the old way using _--link_ was (still is) very simple, but has a scalability problem. When you "--link" two containers, what docker did was to add a line in the _/etc/hosts_ file in one of the containers pointing the other. If you need all containers to see each other, you'll probably end with a messed up _/etc/hosts_ files. Networking comes to solve this and adds the chance to create a complex network topology. And this is way far from what you need to know if you are _starting_ with Docker.

There are several types (drivers) of networks in docker:




  * host - a network using the interface of the host. Thus, it could collide with your own ports
  * bridge - default network. Docker creates this interface (if there's no network) and connect all the containers here
  * none - assigned to containers that you want to say explicitely you don't want to connect to any network
  * overlay - to have a network of several containers in different hosts



In this post, we're only seeing the bridge network, because it's the most similar to what we've done in the previous posts.

Apart from having those types, docker created 3 networks:


  * host - of type host
  * bridge - of type bridge
  * null - of type none



I won't judge the criteria to name things (even if they are confusing as hell). As it was with volumes, with the networks we can do several operations:



    # create a new network
    $ docker network create my_network

    # list the available networks
    $ docker network ls
    NETWORK ID          NAME                DRIVER
    afce7261e0ab        bridge              bridge
    23b60d6f34a4        none                null
    709efbcc9ef5        host                host
    c4c93e560f13        my_network          bridge

    # connect a container to a network
    $ docker network connect my_network my_container

    # inspect a network
    $ docker network inspect my_network
    [
        {
            "Name": "my_network",
            "Id": "c4c93e560f13d50c9a031bae9c0acff5770acd3120abb7997dc88e284d97f3ea",
            "Scope": "local",
            "Driver": "bridge",
            "IPAM": {
                "Driver": "default",
                "Config": [
                    {}
                ]
            },
            "Containers": {
                "8e2762d6d9b6e32895e325f46333ea165fd99e5373af59240109d20f1243a35d": {
                    "EndpointID": "1c8d44f2d90efb10eefcece32c156f2cbef299394e9cdb22e02a064e7b758ec2",
                    "MacAddress": "02:42:ac:12:00:02",
                    "IPv4Address": "172.18.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {}
        }
    ]

    # disconnect a container from a network
    $ docker network disconnect my_network my_container

    # delete a network
    $ docker network rm my_network




These are the basic options in the command line. But network and volumes show all their potential thanks to the _docker-compose.yml_ "version 2".



## New docker-compose syntax



docker-compose now provides a way to create _docker-compose.yml_ files that can reflect all these new citizens. The old docker-compose.yml files will work as before; to use the new version, I'm going to create a new file called _docker-compose-2.yml_ and the examples will assume it. It's necessary to set the version at the beginning of the file:



    version: "2"




Our previous keys now are under a new first-level key:



    version: "2"
    services:
        myapp-postgres:
            image: postgres:9.4.5
            container_name: myapp-postgres
            environment:
                - "POSTGRES_DB=myapp"




After saving it, we can run it:



    # as usual
    (docker) $ docker-compose -f docker-compose-2.yml up -d

    # and see the container running
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    33df5f38934d        postgres:9.4.5      "/docker-entrypoint.s"   4 minutes ago       Up 4 minutes        5432/tcp            myapp_postgres

    # see that docker has created automatically a new network
    $ docker network ls
    NETWORK ID          NAME                DRIVER
    95579ac5ea2e        docker_default      bridge

    # and see that our container is in this new network
    $ docker inspect myapp_postgres
    [...]
    "Networks": {
        "docker_default": {
            "IPAMConfig": null,
            "Links": null,
            "Aliases": [
                "myapp_postgres",
                "cae577804a"
            ],
    [...]




Not bad! So we know how to set the version and the "new" services. We can also create networks in the _docker-file-2.yml_:



    version: "2"
    services:
        myapp-postgres:
            image: postgres:9.4.5
            container_name: myapp-postgres
            environment:
                - "POSTGRES_DB=myapp"
            networks:
                - my-other-network

    networks:
        my-other-network:
           driver: bridge




Try to run it and do again the same checks as above. Note that the new network has an automatic prefix "docker_" (because I executed it from a directory called "docker"):



    $ docker network ls
    NETWORK ID          NAME                      DRIVER
    67855151bf23        docker_my-other-network   bridge




In the first part of the post, we saw that we could create networks directly with _docker_ and assign them our names. So we could create our networks with _docker_ and connect them to our containers in the _docker-compose-2.yml_. This is not even a recommendation: just my thoughts playing with the new features. If you want to do this, you'll need a slightly different docker-compose file:



    version: "2"
    services:
        myapp-postgres:
            image: postgres:9.4.5
            container_name: myapp-postgres
            environment:
                - "POSTGRES_DB=myapp"
            networks:
                - my_network
                - my-other-network

    networks:
        my-other-network:
            driver: bridge
        my_network:
            external:
                name: my_network # we created it at the beginning of this post




If we run it and inspect our container, we can see that it's in the two networks we just configured. yay! Finally, creating and setting volumes is easy compared to networks:



    version: "2"

    services:
        myapp-postgres:
            image: postgres:9.4.5
            container_name: myapp-postgres
            environment:
                - "POSTGRES_DB=myapp"
            networks:
                - my_network
                - my-other-network
            volumes:
                - my-other-volume:/opt/my-other-volume
                - my_volume:/opt/my_volume


    networks:
        my-other-network:
            driver: bridge
        my_network:
            external:
                name: my_network # we created it at the beginning of this post

    volumes:
        my-other-volume:
            driver: local
        my_volume:
            external:
                name: my_volume # we created it at the beginning of this post




You can see the same behaviour as network in the configuration file. Exercise: How would you check everything went as expected?

Congratulations! You've gone too far in this unexplored land. Now you feel like Nellie Bly or Shackelton. You deserve it.

The new features are great, but as a newbie, I needed an extra effort to understand everything. I'm aware that this post is not as beginner friendly as the previous. Probably this is also because my lack of experience in this new features. Anyway, feel free to comment with questions or suggestions or clarifications requests and I'll do my best. I hope you enjoyed and learnt something with this posts. :D

Happy hacking!


