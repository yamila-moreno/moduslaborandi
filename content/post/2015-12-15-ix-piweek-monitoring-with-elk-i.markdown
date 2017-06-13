---
author: yamila
comments: true
date: 2015-12-15 16:00:36+00:00
slug: ix-piweek-monitoring-with-elk-i
title: 'IX PIWEEK: monitoring with ELK (I)'
wordpress_id: 1013
tags:
- elk
- kaleidos.net
- motivation
- my modus
- piweek
---

Yesterday started the [Piweek IX](http://piweek.com), 9 times showing what [a small company](http://kaleidos.net) can do about innovation. Twice a year. You can follow us on [twitter](http://twitter.com/_piweek_).

For this edition, I left aside my tradicional taste for Arduino and wanted to try a well known technology for monitorization (ELK). And I was very happy to know that my colleague [Alex](http://twitter.com/_superalex_) wanted to be part of the experiment.

As usual, I'm trying to build some useful information out from my learning process, so here you have a serie of posts about how to run [ELK](https://www.elastic.co/downloads) (Elasticsearch, Logstash, Kibana).

![piweek-logo](/images/2015/12/piweek-logo.png)

<!-- more -->

This post covers a brief explanation of each component of ELK and the instructions to have it up&running; in its simpliest mode. I've chosen avoid the installation process and use an amazing [docker image]().



## Assumptions



This tutorial assumes that you work fine with the command line, Python and its ecosystem are not new to you, and you know basic concepts about logs. All the commands work in (my) ArchLinux, you may change commands to adapt to your operating system.



## What's ELK?



**Logstash**: ([site](https://www.elastic.co/products/logstash)) is an application which reads several log sources, applies transformations (filters) and returns original logs with the transformations.
**Elasticsearch**: with [Elasticsearch](https://www.elastic.co/products/elasticsearch) we store the logs (after Logstash transformation); besides it's a search engine, so we can create complex queries to filter the documents.
**Kibana**: ([site](https://www.elastic.co/products/kibana)) is a front-web application to visualize those logs in a fancy dashboard, built with different visual components.

![elastic-logo](/images/2015/12/elastic-logo.png)



## Docker



**Installation**

We could install the 3 components from scratch, and there are many tutorials about this topic, but the goal of this specific tutorial is focused on the usage, so we're gonna use a docker image, with almost everything ready.

Install docker (system package) and start the service



    yami $ sudo pacman -S docker
    yami $ sudo systemctl start docker




After installation, we add our user to the group _docker_



    yami $ sudo usermod -aG docker yami




Now, create a python3 virtualenv, where we're going to install _docker-compose_



    yami $ mkvirtualenv monitoralia
    (monitoralia) yami $ pip install docker-compose




Clone the repository (thanks to [deviantony](https://github.com/deviantony)):



    yami $ git clone https://github.com/deviantony/docker-elk.git
    yami $ ls docker-elk
    LICENSE  README.md  docker-compose.yml  elasticsearch  kibana  logstash




After installation, we're going to review some configurations in _docker-compose.yml_ file. In this file, we set the options to run the services.

**Elasticsearch configuration**




    elasticsearch:
      image: elasticsearch:latest
      command: elasticsearch -Des.network.host=0.0.0.0
      ports:
        - "9200:9200"
        - "9300:9300"








  * use the latest image
  * with _command_ we configure the parameters to run the service
  * ports: we map our local ports with ports in docker container



**Logstash configuration**




    logstash:
      image: logstash:latest
      command: logstash agent --config /etc/logstash/conf.d/
      volumes:
        - ./logstash/config:/etc/logstash/conf.d
        - ./taiga-logs:/var/log/taiga
      ports:
        - "5000:5000"
      links:
        - elasticsearch


  * use the latest image
  * with `command` we configure to run logstash reading configuration files in this folder (_/etc/logstash/conf.d/_)
  * volumes: we need to configure in docker the different folders that we'll use in logstash
    * on one hand, configuration file. Here we configure where are the sources for the logs, we create transformations and the path to send the logs to their corresponding storage.
    * on the other hand, we need to make docker know where are we storing logs. In our case, we see just a folder for logs
  * ports: we map our local ports with ports in docker container
  * links: here we say that we are going to need connection from logstash to elasticsearch



**Kibana configuration**




    kibana:
      build: kibana/
      volumes:
        - ./kibana/config/kibana.yml:/opt/kibana/config/kibana.yml
      ports:
        - "5601:5601"
      links:
        - elasticsearch


  * instead of using an existing image, we're going to build a new one
  * we set the volumes where kibana will find its configuration files
  * ports: we map our local ports with ports in docker container



**Run docker**

Once we've done previous steps, we can launch docker:




    (monitoralia) yami $ docker-compose up




Now you can go to **http://localhost:5601** and you'll find the front web of kibana; you'll see tons of options with few sense right now. We'll see more detail in the following posts.

![kibana-screenshot](/images/2015/12/kibana-screenshot.png)

