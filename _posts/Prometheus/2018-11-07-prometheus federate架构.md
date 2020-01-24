



layout: post
title:  "prometheus federate 架构"
categories: "prometheus"
tags: "prometheus"
author: "songzhx"
date:   2018-11-07



> prometheus fedarate 架构介绍：
>
> 官方宣称单个Prometheus Server可以轻松的处理数以百万的时间序列。同时应对更大的集群，prometheus同时可以自由扩展，处理更多的数据。这个自由扩展的架构就是federate架构。



## 1.数据中心 或者 功能分区的形式

### （1）主机配置

**promethues配置：**

| ip           | 角色   | 名字             | 地址             |
| ------------ | ------ | ---------------- |---------------- |
| 10.142.78.13 | master | faderate         | /usr/local/prometheus2.3.2 |
| 10.142.78.12 | slave1 | prometheus       | /usr/local/prometheus2.3.2 |
| 10.142.78.14 | slave2 | prometheus-78.14 | /usr/local/prometheus2.3.2 |

**slave 10.142.78.12  收集metrics的节点列表：**

| ip           | 地址                     | 收集内容    |
| ------------ | ------------------------ | ----------- |
| 10.142.78.22 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.23 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.24 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.25 | /usr/local/node_exporter | 主机metrics |


**slave 10.142.78.14  收集metrics的节点列表：**
| ip           | 地址                     | 收集内容    |
| ------------ | ------------------------ | ----------- |
| 10.142.78.30 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.31 | /usr/local/node_exporter | 主机metrics |

**具体的架构图：**

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczkynkqj30q10cnmxc.jpg)

###（2）配置文件及其启动

master配置 

```yaml
cat  prometheus.yml

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'federate'
    scrape_interval: 15s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'
        - '{__name__=~"node.*"}'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['10.142.78.12:9090','10.142.78.14:9090']

```


slave1配置


```yaml
cat  prometheus.yml

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['10.142.78.12:9090','10.142.78.22:9100','10.142.78.23:9100','10.142.78.24:9100','10.142.78.25:9100']

```


slave2配置

```yaml
cat  prometheus.yml

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus-78.14'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['10.142.78.14:9090','10.142.78.30:9100','10.142.78.31:9100']

```



**启动指令**

```shell
promethus 启动指令

nohup ./prometheus --config.file=prometheus.yml  >>/dev/null 2>&1 &
```



```shell
node_exporter
 nohup ./node_exporter  >>/dev/null 2>&1 &

```



###（3）收集结果


**通过slave-78.12查看负责metrics （10.142.78.22-25）**

![IM截图2018110716330](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczm2p3mj30f90kcmy4.jpg)

**通过slave-78.14查看负责metrics（10.142.78.30-31）**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczmulr9j30gv0hw74t.jpg)

**通过master-78.13查看全局metrics（10.142.78.22-25,10.142.78.30-31）**



![IM截图2018110716331](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcznrubdj30ji0nfmye.jpg)





## 2.水平扩展（同一个job内扩展）



###（1）主机配置

**新增的promethuse主机配置：**

| ip           | 角色   | 名字    | job名字 | 地址                       |
| ------------ | ------ | ------ | -----  | -------------------------- |
| 10.142.78.25 | slaves | slaves | slaves | /usr/local/prometheus2.3.2 |
| 10.142.78.26 | slave1 | slave1 | slave-job | /usr/local/prometheus2.3.2 |
| 10.142.78.27 | slave2 | slave2 | slave-job | /usr/local/prometheus2.3.2 |

**10.142.78.26收集主机列表：**

| ip           | 地址                     | 收集内容    |
| ------------ | ------------------------ | ----------- |
| 10.142.78.16 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.17 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.18 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.19 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.20 | /usr/local/node_exporter | 主机metrics |




**10.142.78.27收集主机列表：**

| ip           | 地址                     | 收集内容    |
| ------------ | ------------------------ | ----------- |
| 10.142.78.16 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.17 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.18 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.19 | /usr/local/node_exporter | 主机metrics |
| 10.142.78.20 | /usr/local/node_exporter | 主机metrics |

### （2）配置文件及其启动



**10.142.78.25 slaves主机**

```yaml
cat promethues.yml


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
  - job_name: slaves
    honor_labels: true
    metrics_path: /federate
    params:
      match[]:
        - '{__name__=~"^slave:.*"}'   # Request all slave-level time series
        - '{__name__=~"job:.*"}'
        - '{__name__=~"node.*"}'

    static_configs:
      - targets:
        - 10.142.78.36:9090
        - 10.142.78.37:9090


```

**10.142.78.36 slave主机**

```yaml
cat promethues.yml

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
  external_labels:
    slave: 1  # This is the 1nd slave. This prevents clashes between slaves.

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

  - job_name: slave_job
    # Add usual service discovery here, such as static_configs
    static_configs:
      - targets:
        - 10.142.78.16:9100
        - 10.142.78.17:9100
        - 10.142.78.18:9100
        - 10.142.78.19:9100
        - 10.142.78.20:9100
    relabel_configs:
    - source_labels: [__address__]
      modulus:       2   # 4 slaves
      target_label:  __tmp_hash
      action:        hashmod
    - source_labels: [__tmp_hash]
      regex:         ^0$  # This is the 2nd slave
      action:        keep

```

**10.142.78.37 slave主机**



```yaml
cat promethues.yml

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
  external_labels:
    slave: 2  # This is the 2nd slave. This prevents clashes between slaves.

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

  - job_name: slave_job
    # Add usual service discovery here, such as static_configs
    static_configs:
      - targets:
        - 10.142.78.16:9100
        - 10.142.78.17:9100
        - 10.142.78.18:9100
        - 10.142.78.19:9100
        - 10.142.78.20:9100
    relabel_configs:
    - source_labels: [__address__]
      modulus:       2   # 4 slaves
      target_label:  __tmp_hash
      action:        hashmod
    - source_labels: [__tmp_hash]
      regex:         ^1$  # This is the 2nd slave
      action:        keep

```



### （3）收集结果



####（a)节点发现

**78.36跟78.37是负载均衡形式**

78.36的自动发现节点：

![8.36结](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczobvl6j31050ej40h.jpg)

78.37自动发现节点

![8.37结](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczp7eu3j312y0eytal.jpg)



#### （b)查询结果-折线图

**具体的折线图展示：**

**78.36的折线图：**

![8.36折线](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczr3qqlj30vw0k0jsh.jpg)



**78.37的折线图：**

![8.37折线](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczs0u8vj30zk0jrwfr.jpg)



78.35的折线图：

![8.35折线](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fczt0qtnj313y0l90uj.jpg)







##  3.更复杂的扩展

最后一种扩展可以结合第一种+第二种方式。







