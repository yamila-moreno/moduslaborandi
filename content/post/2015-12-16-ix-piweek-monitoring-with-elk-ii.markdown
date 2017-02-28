---
author: yamila
comments: true
date: 2015-12-16 12:20:33+00:00
slug: ix-piweek-monitoring-with-elk-ii
title: 'IX PIWEEK: monitoring with ELK (II)'
wordpress_id: 1028
categories:
- Open source
- Tutorials
tags:
- elk
- kaleidos.net
- logstash
- motivation
- my modus
- open source
- piweek
---

Previously on IX piweek: [Using docker to have an ELK environment up & running](http://moduslaborandi.net/ix-piweek-monitoring-with-elk-i/). In this post we are going to check in more detail how to configure **logstash**.

![logstash](/images/2015/12/logstash.png)

<!-- more -->

The first thing we want to do (as we saw in the previous post) is to tell docker where our configuration files are.

    volumes:
        - ./logstash/config:/etc/logstash/conf.d


In this directory, we may have one or more config files. In our example, we have several:

    (monitoralia) yami $ ls logstash/config
    logstash.conf  output.conf  taiga-logs.conf


  * _taiga-logs.conf_ sets taiga.io logs as data source, and applies certain filters.
  * _logstash.conf_ sets tcp requests in port 5000 as data source (Cuidado! this port should be mapped in docker-compose.yml).
  * _output.conf_ sets where are we putting all resultant documents.


**Note**: it seems that logstash concats all config files (instead of merging them). If you have several **outputs**, logstash executes them all, and if they are duplicated, we'll see duplicated logs in our panel. So, in order to organize or logs, we're creating a specific config file for **outputs**.



## Inputs



Apart from **output**, for each data source, we want to configure:




  * **input** the data source itself
  * **filters** transformations over our data

We can configure many different [inputs](https://www.elastic.co/guide/en/logstash/current/input-plugins.html), and among them, we're gonna highlight:




  * _files_: we're chosing this input to collect logs of an application
  * _tcp_: we're using this input to test logstash during this tutorial



With this configuration, we can make a concept test. Launch the containers with _docker-compose up_, and check http://localhost:5601. At this moment, it's needed to have at least a registry in logstash before continuing; so open a new terminal and type the following:



    yami $ echo "hello, world" >> hello
    yami $ nc localhost 5000 < hello




Go back to the browser. Kibana complains about an **index pattern**. So go to "settings">"indexes">"configure an index pattern" and create the default one  (@timestamp). No we can go to the _Discover_ tab and there, we see the log we just added. YAY!!

Now, we're going to add more interesting logs (taiga.io for instance). First, we make them available through the container (in file _docker-compse.yml_)::



    volumes:
        - ./taiga-logs:/var/log/taiga




After that, we make those files an **input** (of type "file") in our _logstash/config/logs-taiga.conf_:



    nput {
        file {
            type => "nginx"
            start_position => "beginning"
            path => [ "/var/log/taiga/*.log" ]
        }
    }




Now we stop _gracefully_ (Ctrl+C) and run again:



    ^CGracefully stopping... (press Ctrl+C again to force)
    Stopping dockerelk_logstash_1 ... done
    Stopping dockerelk_kibana_1 ... done
    Stopping dockerelk_elasticsearch_1 ... done
    dockerelk_elasticsearch_1 exited with code 143
    dockerelk_logstash_1 exited with code 0
    dockerelk_kibana_1 exited with code 137
    (monitoralia) yami $ docker-compose up




If we check _Discover_ tab in Kibana, we have a bunch of logs!! Great!! :D Take a look to the registries in Kibana; we'll come back on this later.

<figure>
  <img src="/images/2015/12/logs-sin-transformaciones.png" width="800px"
       alt="Logs sin transformaciones" />
  <figcaption>Logs sin transformaciones</figcaption>
</figure>


## Filters



It's the perfect moment to apply some **filters**, so the logs are modified and more adecuataed for visualization. In the official documentation we have a list of [all the filters](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) that we can use. We are going to see how they work with some examples.

If we check (like now, yes!) any registry, we can see field **message** is just a long string:



    {
      "_index": "logstash-2015.12.16",
      "_type": "nginx",
      "_id": "AVGqM3SevFw0gELjkkJq",
      "_score": null,
      "_source": {
        "message": "1.1.1.1 - - [14/Dec/2015:10:06:52 +0100] \"OPTIONS /api/v1/issues/custom-attributes-values/1 HTTP/1.1\"
                    200 31 \"https://tree.taiga.io/project/xxx/issue/339\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1)
                    AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36\"",
        "@version": "1",
        "@timestamp": "2015-12-16T09:51:16.501Z",
        "host": "41f6f2dfefde",
        "path": "/var/log/taiga/api-access_ws02.log",
        "type": "nginx"
      },
      "fields": {
        "@timestamp": [
          1450259476501
        ]
      },
      "sort": [
        1450259476501
      ]
    }




Let's apply the first filter, a very simple one, which converts this string in a easier to use json. Edit file _logstash/config/logs-taiga.conf_ and add:



    filter {
        grok {
           match => [ "message" , "%{COMBINEDAPACHELOG}+%{GREEDYDATA:extra_fields}"]
           overwrite => [ "message" ]
        }
    }




**Note** If we're using the same logs, and we are, we should delete data in elasticsearch before launching each time. We have two options:

  * Enter the elasticsearch container and delete indexes. You'll need _elasticsearch-curator_ (install with pip) and run _curator delete indices --all-indices_
  * Delete the container completely (not the image!) with _docker-compose rm_ (delete the 3 containers)



For this tutorial, we use the second option; so we stop docker, delete the container and launch again with _docker-compose up_. Now go and check our panel in Kibana. We chose the first log and its json representation. If everything went well, we see **message** field is now a richier JSON:



    {
      "_index": "logstash-2015.12.16",
      "_type": "nginx",
      "_id": "AVGqfwaMVEmqhbf6ULUs",
      "_score": null,
      "_source": {
        "message": "1.1.1.1 - - [14/Dec/2015:10:06:52 +0100] \"OPTIONS /api/v1/issues/custom-attributes-values/1 HTTP/1.1\"
                    200 31 \"https://tree.taiga.io/project/xxx/issue/339\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1)
                    AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36\"",
        "@version": "1",
        "@timestamp": "2015-12-16T11:13:50.161Z",
        "host": "749c1f979a14",
        "path": "/var/log/taiga/api-access_ws01.log",
        "type": "nginx",
        "clientip": "1.1.1.1",
        "ident": "-",
        "auth": "-",
        "timestamp": "14/Dec/2015:10:06:52 +0100",
        "verb": "OPTIONS",
        "request": "/api/v1/issues/custom-attributes-values/1",
        "httpversion": "1.1",
        "response": "200",
        "bytes": "31",
        "referrer": "\"https://tree.taiga.io/project/xxx/issue/339\"",
        "agent": "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36\""
      },
      "fields": {
        "@timestamp": [
          1450264430161
        ]
      },
      "sort": [
        1450264430161
      ]
    }




Victory!! This deserves a gif!

![](https://media.giphy.com/media/xTk9ZTMoDb2lwAMlry/giphy.gif)

Well, it's time to add a new transformation; we're using filter **geoip**, which derives a geographic location from an IP (note the IP we calculated in the previous step, in the field **clientip**). So, we add to our filters:



    filter {
        grok { ... }
        geoip {
            source => "clientip"
            target => "geoip"
            add_tag => [ "nginx-geoip" ]
        }
    }




Stop the docker, delete elasticsearch container and launch again the system (business as usual ;-)). If you review now any registry, we find a new **geoip** field, with this information:



    "geoip": {
      "ip": "1.1.1.1",
      "country_code2": "ES",
      "country_code3": "ESP",
      "country_name": "Spain",
      "continent_code": "EU",
      "latitude": 40,
      "longitude": -4,
      "location": [
        -4,
        40
      ]
    },




This is pretty cool, and we'll use it later. Besides, we're adding more filters: very simple to understand if you have the [reference](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) at hand:



    filter {
        grok {
            match => [ "message" , "%{COMBINEDAPACHELOG}+%{GREEDYDATA:extra_fields}"]
            overwrite => [ "message" ]
        }

        mutate {
            convert => ["response", "integer"]
            convert => ["bytes", "integer"]
            convert => ["responsetime", "float"]
        }

        geoip {
            source => "clientip"
            target => "geoip"
            add_tag => [ "nginx-geoip" ]
        }

        date {
            match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
            remove_field => [ "timestamp" ]
        }

        useragent {
            source => "agent"
        }
    }





Ok! Well done! Play with the filters and take a rest. In next post, we'll see elasticsearch and eventually we'll learn how to create fancy dashboards to visualize our data. Stay tuned!

