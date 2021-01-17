---
title: ' Anomaly Detection with Docker, Filebeat, Kafka, ELK Stack and Machine Learning (Part -1)'
date: 2020-10-25
permalink: /posts/2020/10/blog-post-5/
tags:
  - Apache Kafka
  - Elastic Search, Logstash, Kibana
  - Machine Learning
  - Filebeat
  - Docker
---

### What is ELK stands for?

The ELK stack is combination of multiple open source products. These are develpoed and well mainted by a company called Elastic.
Lets breakdown what ELK stands for:

* Elasticsearch - It lets you store, search and analyze data with scale
* Logstash - It filters parse each event, identify named fields to build structure, and transform them to converge on a common format for more powerful analysis
* Kibana - visualize data from diverse source with stunning dashboard, manage your deployment in a single UI.
* Filebeat - Lightweight shipper for logs . Whether you’re collecting from security devices, cloud, containers, hosts, or OT, Filebeat helps you keep the simple things simple by offering a lightweight way to forward and centralize logs and files


As i am going to setup lot of products here, i will break down the blog into three parts. First, in part 1 we will discuss about the architecture and how to make the architecture ready. In the part 2, we will see how we can integrate ML model and in the final part we will do all of these together in AWS environment. 

Lets talk about part 1, we will start with the architecture breakdown then followed by a docker container configurations along with docker compose for each products. Finally, we will see how the architecture reads data and process through each components of the pipeline and visualize in Kibana dashboard and monitor. 



## The Architecture:

![Elk-kafka Architecture](/images/elk-archi.png "System Architecture")


To illustrate, the above figure represents the whole pipeline except the ML implementation. The first components Filebeat will read the log from any source then it will send the logs to the producer of the Kafka, the logstash will read the data from kafka broker then make some trsnformation or modifications followed by sending it to the Elsaticsearch. Finally Kibana will get the data from Elsaticsearch. 


## Start the pipeline

### Necessary configuration changes:

* The log file location must be configured in docker-compose.yml file. Here is the filebeat docker compose:

```
  # Filebeat Docker Image
  filebeat:
    image: "docker.elastic.co/beats/filebeat:5.4.3"
    command: filebeat -e -strict.perms=false
    networks:
      - kafkanet
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - ./log:/mnt/log:ro
```


The "volumes:" argument needs to set according the file path in docker container file system. 

* The logstash.conf file in logstash folder must be configure for kafka and the filter to transform the log data. For example, the given logstash.conf file contains following kafka topics which i have created while running the kafka container to test the pipeline at the beginning. 

Logstash.conf file consists of several block:

1. input block to get the data from specific service like kafaka.

```
input {
  kafka {
    bootstrap_servers => "kafkaserver:9092"
    topics => ["sit.catalogue.item","uat.catalogue.item"]
    auto_offset_reset => "earliest"
    decorate_events => true
  }
}
```

2. filter block to filter the data 

#### Filter Plugin. A filter plugin performs intermediary processing on an event.
#### Ref Link - https://www.elastic.co/guide/en/logstash/current/filter-plugins.html

```
filter {
  json {
    source => "message"
  }
  mutate {
    remove_field => [
      "[message]"
    ]
  }
  if (![latency] or [latency]=="") {
    mutate {
      add_field => {
        latency => -1
      }
    }
  }
  date {
    match => [ "time_stamp", "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'" ]
    timezone => "Europe/London"
    target => [ "app_ts" ]
    remove_field => ["time_stamp"]
  }
  if ([@metadata][kafka][topic] == "uat.catalogue.item") {
    mutate {
      add_field => {
        indexPrefix => "uat-catalogue-item"
      }
    }
  }else if ([@metadata][kafka][topic] == "sit.catalogue.item") {
    mutate {
      add_field => {
        indexPrefix => "sit-catalogue-item"
      }
    }
  }else{
    mutate {
      add_field => {
        indexPrefix => "unknown"
      }
    }
  }
}
```

3. output block to send the data to elasticsearch

#### An output plugin sends event data to a particular destination. Outputs are the final stage in the event pipeline.
#### Ref Link - https://www.elastic.co/guide/en/logstash/current/output-plugins.html

```
output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "%{[indexPrefix]}-logs-%{+YYYY.MM.dd}"
  }
}
```

All of the endpoints like hostname and port should be configure according to your preference. 

* Finally we can up the docker-compose.yml file which will trigger all the service running and the pipeline will start working. Then you can visit the elasticsearch dashboard with the corresponding hostname:port to see the data. 

N.B: All the docker compose and configuration files are uploaded in my github repo. Please visit and use the repo as you want. If any problem arise please create a isse and do not forget to start the repor if it helps you someway.

