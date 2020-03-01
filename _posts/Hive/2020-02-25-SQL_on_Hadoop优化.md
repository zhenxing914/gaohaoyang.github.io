## 常见的sql on hadoop

1. hive 

2. Impala : 需要大量内存。 同样适用metastore，

3. presto

4. Drill

5. Phoenix： 使用SQL查询Hbase。

6. Spark SQL

metaStore：（hive、impala、presto、SparkSQL）框架之间是共享元数据信息 ，创建的一些表是可以共用的。



## 1. 优化-架构

### 1. 分表（降IO）
1. 场景

    每分钟2亿条数据、500个作业访问这个大表
 2. 解决方案

    将常用字段剥离出来，构成一张表。
### 2.分区表 partition
1. 场景

   用户日志，一年的数据放在一张表中，数据量会很大

2. 解决方案

   1. 单级分区、多级分区 （按照天、小时分区）
   2. 静态分区、动态分区
###  3.充分利用中间结果集（降IO）
1. 场景
     1. select a，b，c from xxx where  …
     2. select a，c from xxx  …
     3. select a，d from xxx where  ….
2. 优化（降IO）
     1. create table xxx_temp as select a,b,c,d from xxx;
     2. Select a,b,c from xxx_temp;
###  4.压缩
1. 压缩：使用压缩算法来减少数据的过程。

2. 解压缩：将压缩后的数据恢复成原始数据的过程。

3. 优点:空间减小 IO减少

4. 使用场景：
  1. 输入数据
  2. 中间数据
  3. 输出数据
  
5. Hdfs配置
		 
		 ```shell
		  ./bin/hadoop checknative 验证已经安装哪些压缩工具
	```

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z8jxd1xj31240ckk06.jpg" alt="image-20200225203513530" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z8n0ucuj312c0a8jyq.jpg" alt="image-20200225203449036" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z8pvyzdj312a0botgz.jpg" alt="image-20200225203529355" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z8spnu7j312a0440x8.jpg" alt="image-20200225203729923" style="zoom:50%;" />



## 2. 优化-语法  

### 1. 排序：order by、sort by、 distribute by、cluster by
  1. order by 严格模式必须加上limit，**只会产生一个reducer**。（慎用）
  2. sort by ：**保证每个reducer内部是有序的**。set mapred.reduce.task=3
  3. distribute by: 不是排序，是按照指定的字段将数据分到不同的reduce中。
  4. cluster by：是distributeBy 和 sort by 简写
### 2. 控制输出（reducer、partition、task）数量
1. 控制reducer数量
   1. Reducer等于输出文件个数 
   2. Reducer个数少，则Reducer数据量很大  task数据量多，数据倾斜、耗时时间长
   3. Reducer个数多，则Reducer数据量很小，会产生小文件。
   4.  hive reducer 等价于 spark sql partition(200)
   
> **Tip**
>
> reducer的数量并不是越多越好，我们知道有多少个reducer就会生成多少个文件，小文件过多在hdfs中就会占用大量的空间，造成资源的浪费。如果reducer数量过小，导致某个reducer处理大量的数据（数据倾斜就会出现这样的现象），没有利用hadoop的分而治之功能，甚至会产生OOM内存溢出的错误。使用多少个reducer处理数据和业务场景相关，不同的业务场景处理的办法不同。

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z8wktu9j31220fywul.jpg" alt="image-20200225203832744" style="zoom:50%;" />

### 3. join：普通join、mapjoin
1. Hive中有普通join、mapjoin。
2. set hive.auto.convert.join = false
3. 默认join 一个表一个mapper
4. **map join 不进行shuffle** 

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z92e8l2j30rg0mc0yq.jpg" alt="image-20200225203928602" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z95pabcj30oy0eodkz.jpg" alt="image-20200225203948893" style="zoom:50%;" />

### 4. 执行计划
1. explain extend 

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z99f7byj312201w40q.jpg" alt="image-20200225203855854" style="zoom:50%;" />

### 5. distinct + union all 、union

如果遇到要使用union去重的场景，使用distinct + union all比使用union的效果好。

### 6. 数据倾斜问题

数据倾斜的现象：任务进度长时间维持在99%，只有少量reducer任务完成，未完成任务数据读写量非常大，超过10G。在聚合操作是经常发生。 通用解决方法：``set hive.groupby.skewindata=true;``
将一个map reduce拆分成两个map reduce。

说说我遇到过的一个场景，需用统计某个一天每个用户的访问量，SQL如下：

```sql
select t.user_id,count(*) from user_log t group by t.user_id
```

执行这条语句之后，发现任务维持在99%达到一个小时。后面自己分析user_log表，发现user_id有很多数据为null。user_id为null的数据会有一个reducer来处理，导致出现数据倾斜的现象。解决方法有两种：

1. 通过where条件过滤掉user_id为null的记录。
2. 将为null的user_id设置一个随机数值。保证所有数据平均的分配到所有的reducer中处理。

<img src="/Users/song/Library/Application Support/typora-user-images/image-20200301171407611.png" alt="image-20200301171407611" style="zoom:50%;" />



## 3. 优化-运行方面

### 1. 推测执行
   1. 场景 
     1. 集群中机器的负载是不一样的
     2. 集群中配置是不同的
     3. 数据倾斜

         <img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z9c7zpqj30se0aead2.jpg" alt="image-20200225204034401" style="zoom:50%;" />

    2. 解决方案

   Hadoop采用了推测执行（Speculative Execution）机制，它根据一定的法则推测出“拖后腿”的任务，并为这样的任务启动一个备份任务，让该任务与原始任务同时处理同一份数据，并最终选用最先成功运行完成任务的计算结果作为最终结果。

         <img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8xj29b10j311009wjxb.jpg" alt="image-20200225204104192" style="zoom: 40%;" />


​         
### 2. 并行执行

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z9gwscwj30sa0460td.jpg" alt="image-20200225204210930" style="zoom:50%;" />

![image-20200225204158176](https://tva1.sinaimg.cn/large/0082zybpgy1gc8z9jvkvgj311y08aqa0.jpg)

### 3. JVM重用

<img src="https://tva1.sinaimg.cn/large/0082zybpgy1gc8z9n3n1nj30tq0aan0k.jpg" alt="image-20200225204229089" style="zoom:50%;" />



参考：

reducer个数 ： https://zhuanlan.zhihu.com/p/47037815 