---
layout: post
title:  "flume配置SSL加密+二级flume配置+kafka"
categories: "日志系统"
tags: "flume"
author: "songzhx"
date:   2018-01-23
---



## 1.java环境

java环境目录如下：

/usr/java/jdk1.8.0

## 2.flume机器

| id   | ip           | role   | direction            |
| ---- | ------------ | ------ | -------------------- |
| 1    | 10.142.78.72 | client | /usr/local/flume-1.8 |
| 2    | 10.142.78.73 | client | /usr/local/flume-1.8 |
| 3    | 10.142.78.74 | client | /usr/local/flume-1.8 |
| 4    | 10.142.78.75 | server | /usr/local/flume-1.8 |
| 5    | 10.142.78.76 | server | /usr/local/flume-1.8 |



##3.kafka机器列表

| Hostname                      | IP           | Ruler |
| ----------------------------- | ------------ | ----- |
| NM-304-HW-XH628V3-BIGDATA-089 | 10.142.78.17 | kafka |
| NM-304-HW-XH628V3-BIGDATA-090 | 10.142.78.18 | kafka |
| NM-304-HW-XH628V3-BIGDATA-091 | 10.142.78.19 | kafka |





## 4.相关配置

flume client配置

```shell
vim ./conf/flume-conf-ssl-cluster.properties
```

10.142.78.72-74 配置如下：

(1)**针对单个文件收集的配置**：

```properties

#client
a1.sources=r1
a1.sinks=avroSink1 avroSink2
a1.channels=c1
a1.sinkgroups = g1 

a1.sources.r1.type = exec
a1.sources.r1.command=tail -F /tmp/test.log
a1.sources.r1.channels=c1

#Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity=1000
a1.channels.c1.transactionCapacity = 100

#sink1配置
a1.sinks.avroSink1.type=avro
a1.sinks.avroSink1.channel=c1
a1.sinks.avroSink1.hostname=10.142.78.75
a1.sinks.avroSink1.port=4353
a1.sinks.avroSink1.ssl=true
a1.sinks.avroSink1.trust-all-certs=true
a1.sinks.avroSink1.truststore=/tmp/ssl/truststore.jks
a1.sinks.avroSink1.truststore-type=JKS
a1.sinks.avroSink1.truststore-password=123456
a1.sinks.avroSink1.compression-type=deflate

#sink2配置
a1.sinks.avroSink2.type=avro
a1.sinks.avroSink2.channel=c1
a1.sinks.avroSink2.hostname=10.142.78.76
a1.sinks.avroSink2.port=4353
a1.sinks.avroSink2.ssl=true
a1.sinks.avroSink2.trust-all-certs=true
a1.sinks.avroSink2.truststore=/tmp/ssl/truststore.jks
a1.sinks.avroSink2.truststore-type=JKS
a1.sinks.avroSink2.truststore-password=123456
a1.sinks.avroSink2.compression-type=deflate

#set sink group  
a1.sinkgroups.g1.sinks =avroSink1 avroSink2  
#set failover  
a1.sinkgroups.g1.processor.type = failover  
a1.sinkgroups.g1.processor.priority.k1 = 10  
a1.sinkgroups.g1.processor.priority.k2 = 1  
a1.sinkgroups.g1.processor.maxpenalty = 10000 

```

**(2).收集文件夹内所有的文件**

```properties
##收集匹配/tmp/logs/.*log.*的文件
##positonFile需要创建相应的文件
a1.sources.r1.type = TAILDIR
a1.sources.r1.positionFile = /usr/op/postion/taildir_position.json
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /tmp/logs/.*log.*
a1.sources.r1.fileHeader = true
```



启动指令：

```shell
./bin/flume-ng agent --conf conf --conf-file ./conf/flume-conf-ssl-cluster.properties --name a1 -Dflume.root.logger=INFO,console
```



flume server配置

```shell
vim ./conf/flume-conf-ssl-cluster.properties
```



10.142.78.74 10.142.78.75 机器配置如下：  

```properties
#server
a1.sources=avroSrc
a1.channels=memChannel
a1.sinks=k1
a1.sources.avroSrc.type=avro
a1.sources.avroSrc.channels=memChannel

#Bind to all Interface
a1.sources.avroSrc.bind=10.142.78.76
a1.sources.avroSrc.port=4353
#开启SSL
a1.sources.avroSrc.ssl=true
a1.sources.avroSrc.keystore=/tmp/ssl/keystore.jks
a1.sources.avroSrc.keystore-password=123456
a1.sources.avroSrc.keystore-type=JKS
#开启压缩
a1.sources.avroSrc.compression-type=deflate

a1.channels.memChannel.type=memory
a1.channels.memChannel.capacity=1000
a1.channels.memChannel.transactionCapacity = 100

#set sink to kafka
a1.sinks.k1.type= org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k1.kafka.topic = flume-kafka-msg
a1.sinks.k1.channel=memChannel
a1.sinks.k1.kafka.bootstrap.servers = NM-304-HW-XH628V3-BIGDATA-089:9092,NM-304-HW-XH628V3-BIGDATA-090:9092
a1.sinks.k1.kafka.flumeBatchSize = 20
a1.sinks.k1.kafka.producer.acks = 1
a1.sinks.k1.kafka.producer.linger.ms = 1
a1.sinks.k1.kafka.producer.request.timeout.ms = 300000
a1.sinks.k1.kafka.producer.max.request.size = 5271988
a1.sinks.k1.kafka.producer.compression.type = snappy 

#Describe  the sink
#a1.sinks.loggerSink.type = logger
#a1.sinks.loggerSink.channel=memChannel

```

启动指令：

```shell
./bin/flume-ng agent --conf conf --conf-file ./conf/flume-conf-ssl-cluster.properties --name a1 -Dflume.root.logger=INFO,console
```

