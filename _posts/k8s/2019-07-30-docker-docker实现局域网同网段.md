---
layout: post
title:  "Docker实现局域网同网段"
categories: "DockerAndKubernets"
tags: "docker"
author: "songzhx"
date:   2019-07-30 17:05:00
---



## 1.创建虚拟机

```bash
步骤一：创建网络
$ docker network create -d macvlan  --subnet=192.168.30.0/24  --gateway=192.168.30.254   -o parent=ens33  pub_net


548d79fe04c613d3ca180e8689f2207f71534020bc39566d62d0b5aeb67fc8b5

参数解析：
-d macvlan  加载kernel的模块名
--subnet 宿主机所在网段
--gateway 宿主机所在网段网关
-o parent 继承指定网段的网卡

步骤二：运行容器
$ docker run --net=pub_net --ip=172.16.0.100 -it -d --rm centos:6.7 /bin/bash

参数解析：
--ip 可以指定容器的IP
```

参考：

<https://www.cnblogs.com/zipon/p/6362401.html>

<https://my.oschina.net/jastme/blog/1499403>





## 2.配置ssh服务

```
yum install passwd openssl openssh-server -y
```

安装完成后，启动sshd:

```bash
  /usr/sbin/sshd -D
```

这时报以下错误： 

```bash
[root@ /]# /usr/sbin/sshd 
Could not load host key: /etc/ssh/ssh_host_rsa_key 
Could not load host key: /etc/ssh/ssh_host_ecdsa_key 
Could not load host key: /etc/ssh/ssh_host_ed25519_key
```

执行以下命令解决：

```bash
ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''  
ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''
ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key -N '' 
```

然后，修改`/etc/ssh/sshd_config`配置信息：

- `UsePAM yes`改为`UsePAM no`
- `UsePrivilegeSeparation sandbox`改为`UsePrivilegeSeparation no`

可以用vi改，也可以用下面命令

```bash
sed -i "s/#UsePrivilegeSeparation.*/UsePrivilegeSeparation no/g" /etc/ssh/sshd_config
sed -i "s/UsePAM.*/UsePAM no/g" /etc/ssh/sshd_config
```

修改完后，重新启动sshd

```bash
/usr/sbin/sshd -D
```



参考：

https://www.orchome.com/1303



  



