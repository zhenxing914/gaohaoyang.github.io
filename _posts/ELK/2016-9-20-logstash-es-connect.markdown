---
layout: post
title:  "logstash-ealsticsearch "
categories: "ELK"
tags: "logstash elasticsearch"
author: "songzhx"
date:   2016/9/20 14:13:52  


---

# logstash收集数据存储至elasticsearch #
## 日志文件信息

日志文件
/opt/testmessage.log

    192.3.247.32 GET /index.html 15825 0.043
    192.3.248.42 GET /index.html 15825 0.043

## logstash配置文件

logstash配置文件： out_es_test.conf


```python
input {
	file {
		path => "/opt/testmessage.log"
		type => "nginx"
		start_position => "beginning"
}

}

filter {
  grok {
	match=> { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
}   

output {
	stdout{                #标准输出控制台打印输出信息
      codec =>rubydebug		#ruby格式
	}
   	elasticsearch{
		codec => rubydebug
		hosts  => "127.0.0.1"
		index => "testlogstash"
   }
}
```

## logstash启动

启动logstash：

```python
./bin/logstash -f ./conf/out_es_test.conf  # -f参数 指定logstash配置文件
```

