---
layout: post
title:  "通过group_id查询logstash消费kafka的情况"
categories: "Kafka"
tags: "logstash kafka"
author: "songzhx"
date:   2017-7-7
---

# 通过group_id查询logstash消费kafka的情况



>有时候数据量很大时，不能确定是不是logstash瓶颈还是es瓶颈
>
>此时需要监控logstash消费kafka的情况



### 步骤1：查看需要监控的group_id 

```bash
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server broker1:9092 --list
```



### 步骤2：查看指定group_id消费情况

```shell
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server broker1:9092 --list --describe --group test-consumer-group

GROUP                          TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             OWNER
test-consumer-group            test-foo                       0          1               3               2               test-consumer-group_postamac.local-1456198719410-29ccd54f-0
```



###步骤3:查看总共延迟的情况

```shell
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server NM-304-SA5212M4-BIGDATA-531:9092,NM-304-SA5212M4-BIGDATA-532:9092,NM-304-SA5212M4-BIGDATA-533:9092,NM-304-SA5212M4-BIGDATA-534:9092,NM-304-SA5212M4-BIGDATA-535:9092 --describe --group wzfw7_7 |grep -v GROUP |awk '{sum+= $6 };END {print sum}'
```







