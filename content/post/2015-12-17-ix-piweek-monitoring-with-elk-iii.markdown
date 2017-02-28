---
author: yamila
comments: true
date: 2015-12-17 17:18:14+00:00
slug: ix-piweek-monitoring-with-elk-iii
title: 'IX PIWEEK: monitoring with ELK (III)'
wordpress_id: 1042
categories:
- Open source
- Tutorials
tags:
- elasticsearch
- elk
- kaleidos.net
- logstash
- motivation
- piweek
---

This post covers the basics about elasticsearch API. It's the natural continuation of the previous posts about [running an ELK environment](http://moduslaborandi.net/ix-piweek-monitoring-with-elk-i/) and [configuring Logstash](http://moduslaborandi.net/ix-piweek-monitoring-with-elk-ii/).

![elasticlogo](/images/2015/12/elasticlogo.png)

<!-- more -->

**Elasticsearch**, according to the [official documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html) is: _Elasticsearch is a highly scalable open-source full-text search and analytics engine. It allows you to store, search, and analyze big volumes of data quickly and in near real time_. In the ELK stack, **elasticsearch** is in charge of storing the logs (sent by **logstash**) and providing search features through an API.

If we have our environment _up&running;_, we can check the status of elasticsearch server in a browser going to [http://localhost:9200/_cat/health?v](). The **status** column shows one of three colours (green, yellow or red), thus indicating the health of the service.



## Basic usage



To operate with elasticsearch, we need to create an index. In the previous post, we already created one, and we can confirm it going to [http://localhost:9200/_cat/indices?v]():



    health status index               pri rep docs.count docs.deleted store.size pri.store.size
    yellow open   .kibana               1   1          2            0     11.6kb         11.6kb
    yellow open   logstash-2015.12.16   5   1     752894            0    484.9mb        484.9mb

    yami $ curl -XPUT 'localhost:9200/customer?pretty'
    {
          "acknowledged" : true
    }



If you go again to the list of indexes, you can see the new index _customer_.

Through the API you can add documents to an index, but it's out of the scope of this tutorial. Anyway, you can check [the official documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/_index_and_query_a_document.html) for more information.



## Preparing the environment



Maybe it's interesting to play a bit with the available queries; to do this, we're changing a bit our **logstash** configuration, to end with different indices for different sources. First, we have a new [file with registries](https://raw.githubusercontent.com/bly2k/files/master/accounts.zip) and we add it to volumes in our _docker-compose.yml_:



    logstash:
      image: logstash:latest
      command: logstash agent --config /etc/logstash/conf.d/
      volumes:
        - ./logstash/config:/etc/logstash/conf.d
        - ./logs:/var/log/




Then we change our **logstash** configuration so we can handle different indices:



    yami $ ls logstash/config
    accounts.conf  logstash.conf  taiga-logs.conf
    yami $
    yami $ cat logstash/config/logstash.conf
    input {
        tcp {
            type => "tcp"
            port => 5000
        }
    }
    output {
        if [type] == "tcp" {
            elasticsearch {
                hosts => ["elasticsearch:9200"]
            }
        }
    }
    yami $
    yami $ cat logstash/config/accounts.conf
    input {
      file {
          type           => "json"
          start_position => "beginning"
          path           => ["/var/log/accounts.json"]
      }
    }
    filter {
        if [type] == "account" {
            json {
                source => "message"
            }
        }
    }
    output {
        if [type] == "json" {
            elasticsearch {
                hosts => ["elasticsearch:9200"]
                index => "logstash-customer"
            }
        }
    }
    yami $
    yami $ cat logstash/config/taiga-logs.conf
    input {
      file {
        type           => "nginx"
        start_position => "beginning"
        path           => ["/var/log/api-access*.log"]
      }
    }

    filter {
        if [type] == "nginx" {
            grok {
                match     => {"message" => "%{COMBINEDAPACHELOG}+%{GREEDYDATA:extra_fields}"}
                overwrite => ["message"]
            }

            mutate {
                convert   => {"response" => "integer"}
                convert   => {"bytes" => "integer"}
                convert   => {"responsetime" => "float"}
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
    }

    output {
        if [type] == "nginx" {
            elasticsearch {
                hosts => ["elasticsearch:9200"]
                index => "logstash-taiga"
            }
        }
    }



Take some minutes to understand what's going on with this configuration. Pay attention to the conditions in the config files. Note that the indexes are called "logstash-*"; this is important because logstash needs it to use _geo_point_, which will be useful with Kibana visualizations. It's weird, but [that's life](https://github.com/elastic/logstash/issues/3137). Now, you can stop the service, delete all containers and _docker-compose up_ again.

After the addition of new data and new configuration, we end with these indices, if we check [http://localhost:9200/_cat/indices?v]()



    health status index             pri rep docs.count docs.deleted store.size pri.store.size
    yellow open   .kibana             1   1          1            0      3.1kb          3.1kb
    yellow open   logstash-customer   5   1       2000            0      1.1mb          1.1mb
    yellow open   logstash-taiga      5   1     752894            0    487.5mb        487.5mb






## Performing queries



Ok! Let's play with the elasticsearch API.

**Note**: With Kibana front, you won't need at first to know this API, but it can be useful to customize some visualizations.

All searches are performed against an index, specified in the URL, for instance:



    http://localhost:9200/<strong>logstash-customer</strong>/_search?q=QUERY
    http://localhost:9200/<strong>logstash-taiga</strong>/_search?q=QUERY




Now, we can add some filters and explore the indices; as we've expanded the _message_ field during the **logstash** phase, we have all documents as json with searchable fields. AFAIK, I only can make complex queries passing filters in the body request (in a command line for example), like this:



    yami $ curl -XGET 'http://localhost:9200/_search?size=5&pretty;' -d '
    {
        "query": { "match_all": {} },
        "_source": ["account_number", "balance", "lastname"]
    }'
    yami $ curl -XGET 'http://localhost:9200/logstash-customer/_search?size=10&pretty;' -d '
    {
      "query": {
        "filtered": {
          "query": { "match_all": {} },
          "filter": {
            "range": {
              "balance": {
                "gte": 20000,
                "lte": 30000
              }
            }
          }
        }
      }
    }'




We can also make aggregations:



    yami $ curl -XGET 'http://localhost:9200/logstash-customer/_search?pretty' -d '
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "balance"
          }
        }
      }
    }'




At this point, you know:

* how to set different indices
* how to access the API
* how to perform basic queries

Now you can learn more about [queries](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/search.html) or wait for the next post about visualizing data in Kibana :_)


