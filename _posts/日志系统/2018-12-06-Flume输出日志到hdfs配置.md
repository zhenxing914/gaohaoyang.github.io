---
layout: post
title:  "Flume输出日志到hdfs配置"
categories: "日志系统"
tags: "log 日志系统"
author: "songzhx"
date:   2018-12-06
---


通过flume输出日志存储至HDFS的配置文件如下：



```
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#  http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.


# The configuration file needs to define the sources, 
# the channels and the sinks.
# Sources, channels and sinks are defined per agent, 
# in this case called 'agent'

# list the sources, sinks and channels in the agent
agenttest.sources = CreditAvroSrc
agenttest.sinks = CeditHDFSSink CreditAvroSink
agenttest.channels = CreditFc

# set channels for source
# Bind the source to the channel
agenttest.sources.CreditAvroSrc.channels = CreditFc
agenttest.sinks.CreditHDFSSink.channel = CreditFc
agenttest.sinks.CreditAvroSink.channel = CreditFc

# Describe/configure avro source
agenttest.sources.CreditAvroSrc.type = avro 
agenttest.sources.CreditAvroSrc.bind = 10.142.108.121
agenttest.sources.CreditAvroSrc.port = 4547
agenttest.sources.CreditAvroSrc.threads = 50


# Use a channel which buffers events in memory
agenttest.channels.CreditFc.type = memory
agenttest.channels.CreditFc.keep-alive = 30
agenttest.channels.CreditFc.capacity = 1000000
agenttest.channels.CreditFc.transactionCapacity = 1000

# Describe hdfs  sink
agenttest.sinks.CreditHDFSSink.type = hdfs
agenttest.sinks.CreditHDFSSink.hdfs.path = hdfs://ns3/domain/ns3/ocdc/credit/nginxlogtest/%Y-%m-%d/%H%M
agenttest.sinks.CreditHDFSSink.hdfs.filePrefix = nginxlog
agenttest.sinks.CreditHDFSSink.hdfs.fileSuffix = .dat
agenttest.sinks.CreditHDFSSink.hdfs.inUseSuffix = .tmp
agenttest.sinks.CreditHDFSSink.hdfs.fileType = DataStream
agenttest.sinks.CreditHDFSSink.hdfs.writeFormat = Text
agenttest.sinks.CreditHDFSSink.hdfs.rollInterval = 300
agenttest.sinks.CreditHDFSSink.hdfs.rollSize = 134217728
agenttest.sinks.CreditHDFSSink.hdfs.rollCount = 1000000
agenttest.sinks.CreditHDFSSink.hdfs.round = true
agenttest.sinks.CreditHDFSSink.hdfs.roundValue = 5
agenttest.sinks.CreditHDFSSink.hdfs.roundUnit = minute
agenttest.sinks.CreditHDFSSink.hdfs.batchSize = 1000
agenttest.sinks.CreditHDFSSink.hdfs.callTimeout = 180000
agenttest.sinks.CreditHDFSSink.hdfs.idleTimeout = 60000
agenttest.sinks.CreditHDFSSink.hdfs.closeWritersOnException = true
agenttest.sinks.CreditHDFSSink.hdfs.minBlockReplicas = 1
agenttest.sinks.CreditHDFSSink.hdfs.kerberosPrincipal = ocdc@HADOOP.CHINATELECOM.CN
agenttest.sinks.CreditHDFSSink.hdfs.kerberosKeytab = /usr/local/flume/conf/ocdc.keytab
agenttest.sinks.CreditHDFSSink.hdfs.useLocalTimeStamp = true



#---------------- TO 10.140.16.201:4565 avro ----------------#
# Describe avro  sink
#agent.sinks.CreditAvroSink.type = avro
#agent.sinks.CreditAvroSink.hostname = 10.140.16.201
#agent.sinks.CreditAvroSink.port = 4565
#agent.sinks.CreditAvroSink.batch-size = 9


```

