---
layout: post
title:  "easylog自动添加appName"
categories: "日志系统"
tags: "log 日志系统"
author: "songzhx"
date:   2018-02-28
---



```shell
10.142.102.149

脚本如下：

#!/bin/bash
curl http://10.142.102.149:8200/_cat/indices?v |grep easylog-[a-z] | awk  '{print $3}'  | awk -F '-' '{print $2"-"$3"-"$4"-"$5}' | while read line ; do
 #echo $line 
 query={\"query\":{\"match\":{\"appId.keyword\":\"$line\"}}}
 #curl -XPOST http://10.142.102.149:8200/easylog/app/_search -d '{"query":{"match":{"appId.keyword":"$line"}}}'|grep "appId"
 result=$(curl -XPOST http://10.142.102.149:8200/easylog/app/_search -d $query |grep appId)
 if [ "$result" == "" ] ; then
    insert_dsl={\"appId\":\"$line\",\"appName\":\"$line\"}
    #echo $insert_dsl
    curl -XPOST http://10.142.102.149:8200/easylog/app -d $insert_dsl
 fi
done

```

