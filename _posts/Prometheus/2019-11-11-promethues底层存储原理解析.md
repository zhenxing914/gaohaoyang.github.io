---
layout: post
title:  "promethues底层存储原理解析"
categories: "prometheus"
tags: "prometheus consul"
author: "songzhx"
date:   2019-11-11 16:40:00
---

抓取的原始数据格式：

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u73bryaaj30y40cwjxc.jpg" alt="image-20191111164727313" style="zoom:50%;" />



进一步分析，时序数据可以以key-value的形式进行存储。

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8w5kelcj30y60f87af.jpg" alt="image-20191111164825505" style="zoom:50%;" />

进一步处理，将上面数据处理成以下样式：

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8w87hkij30xs0ceta7.jpg" alt="image-20191111164923531" style="zoom:50%;" />



这样每个lables会对应一个或者过个chunk文件：

chunk文件如下：

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8way557j30wm0fa0vh.jpg" alt="image-20191111165115320" style="zoom:50%;" />



下面我们来分析下具体的index文件的格式：

### 整体格式：

index文件的整体格式如下：

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wgn6c8j30t20usdhv.jpg" alt="image-20191111165430929" style="zoom:50%;" />

### Symbol Table：

记录了每个label，这样可以很好的压缩数据。

例如：

```
status=200
method=get 
```



### Serial:

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wjv4thj30lc09wweo.jpg" alt="image-20191111165726295" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wm8wbej30u00uen0h.jpg" alt="image-20191111165746979" style="zoom:50%;" />

​		这块是真正存储着时序数据之间的关系，上半部分ref(l_i.name),ref(l_i.value)记录着label信息,指向symbol table，下半部分记录着chunk信息，其中c_0.mint对应timestamp最小值，ref(c_0.data)指向chunk的位置。



### Lable index：

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wp33avj314i0osmzd.jpg" alt="image-20191111170107299" style="zoom:67%;" />



value_0是指向Symbol Table表中位置，拼装成如下格式：

```
status=200 method=GET 
status=200 method=POST
```

上面例子：names为2 ， entries =4



### Lable Offset Table：



<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wspomaj30mk0fqgmj.jpg" alt="image-20191111170212353" style="zoom:50%;" />

其中offset是指向Lable Index的偏移量。

主要用于查询name 对应的value组合数量。

例如：name 为 status ， method

此时得出结果：

status=200 method=GET 
status=200 method=POST



### Postings

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wvmcdxj30kw0dut9f.jpg" alt="image-20191111171045365" style="zoom:50%;" />

这个记录了时序数据的索引



### Postings Offset Table

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g8u8wy2gnaj30m20he0ty.jpg" alt="image-20191111171215328" style="zoom:50%;" />

此处记录了，每个lable对应serial之间的关系，是个倒排的关系。

