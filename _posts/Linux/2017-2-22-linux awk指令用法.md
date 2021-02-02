---
layout: post
title:  "linux awk指令用法"
categories: "Linux"
tags: "linux awk"
author: "songzhx"
date:   2017-2-22 12：00：00

---

# linux awk指令用法

```bash
##BEGIN内部语法和c语言语法很相似
##循环将data1/es5.2_data-data12/es5.2_data 的文件夹拥有者改成es：es
sudo awk 'BEGIN{count=13;i=1;while(i<count){ cmd2="chown es:es /data"i"/es5.2_data"; i++; system(cmd2) }}'


##创建文件，并且修改权限
sudo awk 'BEGIN{count=13;i=1;while(i<count){ cmd1="mkdir /data"i"/es5.2_data"; system(cmd1); cmd2="chown es:es /data"i"/es5.2_data";  system(cmd2);i++ }}'


##查看文件更改是否成功
sudo awk 'BEGIN{count=13;i=2;while(i<count){ cmd1="ls -all /data"i""; system(cmd1);i++ }}'
```

```bash
##查看修改后的文件信息
total 28
drwxr-xr-x   4 root root  4096 Feb 22 10:21 .
dr-xr-xr-x. 30 root root  4096 Jan 13 13:51 ..
drwxr-xr-x   2 es   es    4096 Feb 22 10:21 es5.2_data
drwx------   2 root root 16384 Jan 13 11:29 lost+found
total 28
drwxr-xr-x   4 root root  4096 Feb 22 10:21 .
dr-xr-xr-x. 30 root root  4096 Jan 13 13:51 ..
drwxr-xr-x   2 es   es    4096 Feb 22 10:21 es5.2_data
drwx------   2 root root 16384 Jan 13 11:30 lost+found
total 28
drwxr-xr-x   4 root root  4096 Feb 22 10:21 .
dr-xr-xr-x. 30 root root  4096 Jan 13 13:51 ..
drwxr-xr-x   2 es   es    4096 Feb 22 10:21 es5.2_data
drwx------   2 root root 16384 Jan 13 11:30 lost+found
```




## 参考文献

- http://blog.csdn.net/cy_cai/article/details/41908921
- http://www.shencan.net/index.php/2012/09/03/%E5%9C%A8awk%E4%B8%AD%E8%BF%90%E8%A1%8Cshell%E5%91%BD%E4%BB%A4/