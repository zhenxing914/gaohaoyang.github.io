---
layout: post
title:  "flume数据收集存储到ealsticsearch "
categories: "ELK"
tags: "flume elasticsearch"
author: "songzhx"
date:   2016/9/7 15:13:35 


---


# flume1.6收集日志传输到elasticsearch2.3.4存储 #

## 本文的配置
- flume 1.6
- elasticsearch 2.3.4


## eslasticsearch.yml 文件源码

    # ======================== Elasticsearch Configuration =========================
    #
    # NOTE: Elasticsearch comes with reasonable defaults for most settings.
    #   Before you set out to tweak and tune the configuration, make sure you
    #   understand what are you trying to accomplish and the consequences.
    #
    # The primary way of configuring a node is via this file. This template lists
    # the most important settings you may want to configure for a production cluster.
    #
    # Please see the documentation for further information on configuration options:
    # <http://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration.html>
    #
    # ---------------------------------- Cluster -----------------------------------
    #
    # Use a descriptive name for your cluster:
    #
     cluster.name: elasticsearch
    #
    # ------------------------------------ Node ------------------------------------
    #
    # Use a descriptive name for the node:
    #
     node.name: node-72
    #
    # Add custom attributes to the node:
    #
    # node.rack: r1
    #
    # ----------------------------------- Paths ------------------------------------
    #
    # Path to directory where to store the data (separate multiple locations by comma):
    #
    # path.data: /path/to/data
    #
    # Path to log files:
    #
    # path.logs: /path/to/logs
    #
    # ----------------------------------- Memory -----------------------------------
    #
    # Lock the memory on startup:
    #
    # bootstrap.mlockall: true
    #
    # Make sure that the `ES_HEAP_SIZE` environment variable is set to about half the memory
    # available on the system and that the owner of the process is allowed to use this limit.
    #
    # Elasticsearch performs poorly when the system is swapping the memory.
    #
    # ---------------------------------- Network -----------------------------------
    #
    # Set the bind address to a specific IP (IPv4 or IPv6):
    #
     network.host: 10.142.78.72
    #
    # Set a custom port for HTTP:
    #
     http.port: 8200
    #
     transport.tcp.port: 8300
    # For more information, see the documentation at:
    # <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html>
    #
    # --------------------------------- Discovery ----------------------------------
    #
    # Pass an initial list of hosts to perform discovery when new node is started:
    # The default list of hosts is ["127.0.0.1", "[::1]"]
    #
    # discovery.zen.ping.unicast.hosts: ["host1", "host2"]
     discovery.zen.ping.unicast.hosts: ["10.142.78.72:8200", "10.142.78.73:8200"]
    #
    # Prevent the "split brain" by configuring the majority of nodes (total number of nodes / 2 + 1):
    #
    # discovery.zen.minimum_master_nodes: 3
    #
    # For more information, see the documentation at:
    # <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html>
    #
    # ---------------------------------- Gateway -----------------------------------
    #
    # Block initial recovery after a full cluster restart until N nodes are started:
    #
    # gateway.recover_after_nodes: 3
    #
    # For more information, see the documentation at:
    # <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html>
    #
    # ---------------------------------- Various -----------------------------------
    #
    # Disable starting multiple nodes on a single system:
    #
    # node.max_local_storage_nodes: 1
    #
    # Require explicit names when deleting indices:
    #
    # action.destructive_requires_name: true



## flume_es.properties 文件源码##

    # set sources
    agent.sources = tail  
     
    agent.sources.tail.channels = memoryChannel  
      
    agent.sources.tail.type = exec  
      
    agent.sources.tail.command = tail -F  /usr/local/es/tests_logs/logs/song_log.log  
      
    agent.sources.tail.interceptors=i1 i2 i3  
      
    agent.sources.tail.interceptors.i1.type=regex_extractor  
      
    agent.sources.tail.interceptors.i1.regex = (\\w.*):(\\w.*):(\\w.*)\\s  
      
    agent.sources.tail.interceptors.i1.serializers = s1 s2 s3  
      
    agent.sources.tail.interceptors.i1.serializers.s1.name = source  
      
    agent.sources.tail.interceptors.i1.serializers.s2.name = type  
      
    agent.sources.tail.interceptors.i1.serializers.s3.name = src_path  
      
    agent.sources.tail.interceptors.i2.type=org.apache.flume.interceptor.TimestampInterceptor$Builder  
      
    agent.sources.tail.interceptors.i3.type=org.apache.flume.interceptor.HostInterceptor$Builder  
      
    agent.sources.tail.interceptors.i3.hostHeader = host  
      
    # set channel 
    agent.channels = memoryChannel  
      
    agent.channels.memoryChannel.type = memory   
    
    # set sinks
      
    agent.sinks = elasticsearch  
    
    agent.sinks.elasticsearch.type = com.frontier45.flume.sink.elasticsearch2.ElasticSearchSink
    
    agent.sinks.elasticsearch.hostNames=10.142.78.72:8300  
    
    agent.sinks.elasticsearch.channel = memoryChannel  
    
    agent.sinks.elasticsearch.indexType = flume_test_type
      
    agent.sinks.elasticsearch.indexName=flume_test_index
    
    agent.sinks.elasticsearch.clusterName = elasticsearch
    
    agent.sinks.elasticsearch.batchSize=100  
    
    agent.sinks.elasticsearch.serializer=com.frontier45.flume.sink.elasticsearch2.ElasticSearchDynamicSerializer
    
    agent.sinks.elasticsearch.indexNameBuilder = com.frontier45.flume.sink.elasticsearch2.TimeBasedIndexNameBuilder


## flume 执行语句 ##

    ./bin/flume-ng agent -n agent -c conf -f ./conf/flume-es.properties  -Dflume.root.logger=DEBUG,console

## jar文件配置 ##

flume 1.6 不支持ealsticsearch2.3 需要配置ElasticsearchSink2

ElasticsearchSink2 github链接地址：
https://github.com/lucidfrontier45/

 

### ElasticsearchSink2 ###

This is Flume-NG Sink for Elasticsearch >= 2.0. I developed this because the official version does not support Elasticsearch >= 2.0 due to API changes. Hope Flume dev team will fix this soon.

### Requirements ###

Flume-NG >= 1.6

Elasticsearch >= 2.0

### Build ###

Build standard jar by the following command

    $ ./gradlew build

Build fat jar which contains elasticsearch dependencies


    $ ./gradlew assembly

Jar will be generated in build/libs

### Usage ###

1. Append the built jar to Flume's classpath
2. remove guava-*.jar and jackson-core-*.jar in flume's default libs dir. They are outdated and newer version are included in Elasticsearch.
3. set sink type to com.frontier45.flume.sink.elasticsearch2.ElasticSearchSink in flume.conf
4. start flume agent

