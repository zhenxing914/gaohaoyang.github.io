---
layout: post
title:  "ansible_elasticsearch handbook"
categories: "ansible"
tags: "elasticsearch ansible"
author: "songzhx"
date:   2019-09-04 09:03:00

---



## 1.ansible环境配置

```bash
ansible-galaxy install elastic.elasticsearch,7.1.1
```

假如本地已经配置好ansible环境，此步骤可以省略。



## 2.编写playbook.yml

```yml
- hosts: master_node
  roles:
    - role: elastic.elasticsearch
  vars:
  	es_version: "7.1.1"
    es_heap_size: "1g"
    es_config:
      cluster.name: "test-cluster"
      discovery.seed_hosts: "elastic02:9300"
      http.port: 9200
      node.data: false
      node.master: true
      bootstrap.memory_lock: false
    es_plugins:
     - plugin: ingest-attachment

- hosts: data_node_1
  roles:
    - role: elastic.elasticsearch
  vars:
    es_version: "7.1.1"
    es_data_dirs:
      - "/opt/elasticsearch"
    es_config:
      cluster.name: "test-cluster"
      discovery.seed_hosts: "elastic02:9300"
      http.port: 9200
      node.data: true
      node.master: false
      bootstrap.memory_lock: false
    es_plugins:
      - plugin: ingest-attachment

- hosts: data_node_2
  roles:
    - role: elastic.elasticsearch
  vars:
    es_config:
      es_version: "7.1.1"
      cluster.name: "test-cluster"
      discovery.seed_hosts: "elastic02:9300"
      http.port: 9200
      node.data: true
      node.master: false
      bootstrap.memory_lock: false
    es_plugins:
      - plugin: ingest-attachment
```



## 3.添加hosts文件

```yml
[master_node]
192.168.1.2

[data_node_1]
192.168.1.3

[data_node_2]
192.168.1.3
```



## 4.执行playbook

```bash
ansible-playbook -i hosts ./your-playbook.yml
```



