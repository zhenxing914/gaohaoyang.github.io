---
layout: post
title:  "SELinux简易教程"
categories: "linux"
tags: "SELinux"
author: "songzhx"
date:   2017-1-4
---

# SELinux简易教程
>SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，是 Linux历史上最杰出的新安全子系统。在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。SELinux 默认安装在 Fedora 和 Red Hat Enterprise Linux 上。

      ![](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcy1q9jrj30xb0bsgmv.jpg)

虽然SELinux很好用，但是在多数情况我们还是将其关闭，因为在不了解其机制的情况下使用SELinux会导致软件安装或者应用部署失败。



以下就是关闭SELinux的方法

系统版本：centos 6.5 mini



**1、查看selinux状态**

查看selinux的详细状态，如果为enable则表示为开启



```shell
#/usr/sbin/sestatus -v
```

查看selinux的模式

```shell
# getenforce
```



**2、关闭selinux**

2.1:永久性关闭（这样需要重启服务器后生效）

```shell
# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```



2.2:临时性关闭（立即生效，但是重启服务器后失效）



\#设置selinux为permissive模式（即关闭）

```shell
# setenforce 0
```



\#设置selinux为enforcing模式（即开启）

```
# setenforce 1
```



这样就关闭SELinux了，当安装软件遇到问题时可以考虑关闭SELinux再进行安装


## 参考文献

- http://doiido.blog.51cto.com/5503054/1554485