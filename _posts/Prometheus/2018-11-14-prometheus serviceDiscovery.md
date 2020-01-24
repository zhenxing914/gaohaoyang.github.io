---
layout: post
title:  "prometheus 服务自动发现"
categories: "prometheus"
tags: "prometheus"
author: "songzhx"
date:   2018-11-14

---



## 1.基于文件的自动发现

>基于文件的自动发现机制是一个比较通用的自动发现机制。
>
>prometheus以前通过读取<static_config>s配置选项来获取服务列表。 现在通过磁盘监控，发现文件改变则将主机列表进行更新。
>
>



promethues通过监控hosts.yml配置文件来自动发现新加入的节点。

### （1）基本配置

配置文件:

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus-78.14'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    #static_configs:
    #- targets: ['10.142.78.14:9090','10.142.78.30:9100','10.142.78.31:9100']

    file_sd_configs:
    # Patterns for files from which target groups are extracted.
     - files: ['./hosts.yml']
     # Refresh interval to re-read the files.
       refresh_interval: 1m 

```

2个节点的hosts.yml配置

```yaml
[
  {
    "targets": [ "10.142.78.30:9100","10.142.78.31:9100"]
  }
]

```

新增一个节点的hosts.yml配置

```yaml
[
  {
    "targets": [ "10.142.78.30:9100","10.142.78.31:9100","10.142.78.14:9100" ]
  }
]

```



### （2）集群节点视图

hosts.yml包含2个节点时：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcztuy7zj311v09774u.jpg)



hosts.yml添加一个节点后：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczuc3joj312h09qt9f.jpg)