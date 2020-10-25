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

The ELK stack is combination of multiple open source products. These are develpoed and well mainted by a company called Elastic.
Lets breakdown what ELK stands for:

..* Elasticsearch - It lets you store, search and analyze data with scale
..* Logstash - 
..* Kibana - visualize data from diverse source with stunning dashboard, manage your deployment in a single UI.


As i am going to setup lot of products here, i will break down the blog into three parts. First, in part 1 we will discuss about the architecture and how to make the architecture ready. In the part 2, we will see how we can integrate ML model and in the final part we will do all of these together in AWS environment. 

Lets talk about part 1, we will start with the architecture breakdown then followed by a docker container configurations along with docker compose for each products. Finally, we will see how the architecture reads data and process through each components of the pipeline and visualize in Kibana dashboard and monitor. 





