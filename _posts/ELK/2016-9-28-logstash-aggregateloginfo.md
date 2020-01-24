---
layout: post
title:  "logstash 收集多种日志类型转存到ES中"
categories: "ELK"
tags: "logstash"
author: "songzhx"
date:   2016-9-28


---


#logstash收集anacron.log  CROND.log...日志

###日志的目录如下：
![](C:\Users\song\Desktop\tree.png)

    harbor
    |---2016-09-14
    |   |----- anacron.log  
    |   |----- CROND.log
    |   |----- jobservice.log
    |   |----- mysql.log
    |   |----- ...
    |   |----- ...
    |---2016-09-15
    |   |----- anacron.log  
    |   |----- CROND.log
    |   |----- jobservice.log
    |   |----- mysql.log
    |   |----- ...
    |   |----- ...
    |---2016-09-16
    |   |----- anacron.log  
    |   |----- CROND.log
    |   |----- jobservice.log
    |   |----- mysql.log
    |   |----- ...
    |   |----- ...

###logstash具体的配置文件源码如下：

    # anacron.log  CROND.log  jobservice.log  mysql.log  proxy.log  registry.log  run-parts.log  ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/anacron.log"
    		type => "anacron"
    		start_position => "beginning"
    	}
    }
    
    # CROND.log 
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/CROND.log"
    		type => "crond"
    		start_position => "beginning"
    	}
    }
    
    # jobservice.log  mysql.log  proxy.log  registry.log  run-parts.log  ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/jobservice.log"
    		type => "jobservice"
    		start_position => "beginning"
    	}
    }
    
    # mysql.log  proxy.log  registry.log  run-parts.log  ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/mysql.log"
    		type => "mysql"
    		start_position => "beginning"
    	}
    }
       
    #  proxy.log  registry.log  run-parts.log  ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/proxy.log"
    		type => "proxy"
    		start_position => "beginning"
    	}
    }
    
    #  registry.log  run-parts.log  ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/registry.log"
    		type => "registry"
    		start_position => "beginning"
    	}
    }
    
    # run-parts.log  ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/run-parts.log"
    		type => "run-parts"
    		start_position => "beginning"
    	}
    }
    
    #   ui.log
    input {
    	file {
    		path => "/usr/local/logstash/harbor/*/ui.log"
    		type => "ui"
    		start_position => "beginning"
    	}
    }


    #  filter {
    #  grok {
    #  match=> { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
    #   }




    # anacron.log  CROND.log  jobservice.log  mysql.log  proxy.log  registry.log  run-parts.log  ui.log
    # output
    output {
    	if [type] == "anacron"
    	{	
    		elasticsearch{
    			codec => rubydebug
    			hosts  => "192.168.2.10"
    			index => "anacron"
    				 }
    	}
    
    # cron
    	if [type] == "crond"
    	{	
    		elasticsearch{
    			codec => rubydebug
    			hosts  => "192.168.2.10"
    			index => "crond"
    				 }
    	}
    
    }

