---
layout: post
title:  "ES-Hadoop hive读取ES中数据"
categories: "ELK"
tags: "ES-Hadoop hive"
author: "songzhx"
date:   2018-12-13
---

```
hive库名： jt_jtsjzxsjptc_foundation
```

es中数据mapping

```json
{
  "yarn_logaudit20180917": {
    "mappings": {
      "000003": {
        "properties": {
          "appID": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "containerID": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "discription": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "ip": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "logType": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "opTime": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "operation": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "permisions": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "result": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "targetName": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "userName": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```





```shell
1.进入hive客户端 然后添加jar包

hive：ADD  jar    /home/op/es-hadoop/elasticsearch-hadoop-5.2.2.jar

2.创建表：
CREATE EXTERNAL TABLE yarn_logaudit201812 (
 logType STRING,
 ip STRING,
 result STRING,
 opTime STRING,
 operation STRING,
 permisions STRING ,
 targetName STRING,
 containerID STRING,
 discription STRING,
 userName STRING,
 appID STRING
)
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler'
TBLPROPERTIES('es.resource' = 'yarn_logaudit201812*/000003',
              'es.nodes'='10.142.106.84',
              'es.port'='8200',
              'es.index.auto.create' = 'false',
              'es.mapping.names'= 'logtype:logType , optime:opTime , targetname:targetName , containerid:containerID , username:userName , appid:appID') ;

3.查询表数据
select * from yarn_logaudit201812 limit 5;

4.查询表数据总数
select count(*) from yarn_logaudit201812;

执行MR任务需要先配置，job的队列，配置如下：
set mapreduce.job.queuename=root.normal_queues.backstage.yfzx.op;
```



- es.mapping.names : es中字段是驼峰模式，hive中字段都是小写，所以需要配置一个字段映射。
- es.resource : 读模式支持多个index名。



**参考：**

[ES-Hadoop hive使用](https://www.elastic.co/guide/en/elasticsearch/hadoop/5.2/hive.html)





