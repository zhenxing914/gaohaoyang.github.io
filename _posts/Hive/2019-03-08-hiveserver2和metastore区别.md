---
layout: post
title:  "hiveserver2和metastore区别"
categories: "Hive"
tags: "Hive"
author: "songzhx"
date:   2019-03-08 17:50:00
---

## 1. hiveserver2和metastore

**HiveServer2和MetaStore本质上都是Thrift Service**，虽然可以启动在同一个进程内，但不建议这么做。

建议是拆成不同的服务进程来启动

具体可以看代码，看看 HiveServer2到底启动了些什么： 

<https://www.codatlas.com/github.com/apache/hive/master/service/src/java/org/apache/hive/service/server/HiveServer2.java?line=112>

一般来讲，我们认为HiveServer2是用来提交查询的，也就是用来访问数据的。

而MetaStore才是用来访问元数据的。

如果你把两者混了，起在同一个进程内，就会产生你的问题类的疑问。

**CliDriver是SQL本地直接编译，然后访问MetaStore，提交作业，是重客户端。**

**BeeLine是把SQL提交给HiveServer2，由HiveServer2编译，然后访问MetaStore，提交作业，是轻客户端。**

具体写业务脚本两种都行，数据量大的话，建议用CliDriver



## 2. metastore作用

客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。



## 3. beeline的使用方法

``` bash
##启动hiveServer2
启动hiveserver2

## 通过beeline连接hiveServer2
beeline> !connect jdbc:hive2://localhost:10000

```

