# 1. 优化架构

## 1. 分表(降IO)

## 2. 分区

### 2.1 动态分区

```sql

hive.exec.dynamic.partition.mode=strict：该模式下必须指定一个静态分区

hive.exec.max.dynamic.partitions=1000

hive.exec.max.dynamic.partitions.pernode=1000：在每一个mapper/reducer节点允许创建的最大分区数

DATANODE：dfs.datanode.max.xceivers=8192：允许DATANODE打开多少个文件

```

## 3. 充分利用中间结果



## 4. 压缩



# 2. 优化-语法

## 1. 排序

```sql

ORDER BY colName ASC/DESC

hive.mapred.mode=strict时需要跟limit子句

hive.mapred.mode=nonstrict时使用单个reduce完成排序

SORT BY colName ASC/DESC ：每个reduce内排序

DISTRIBUTE BY(子查询情况下使用 )：控制特定行应该到哪个reducer，并不保证reduce内数据的顺序

CLUSTER BY ：当SORT BY 、DISTRIBUTE BY使用相同的列时。
```

## 2. 设置合理的map、reduce数

### 2.1 splitSize的计算方式

**map数的主要决定因素有：**

 input的文件总个数，input的文件大小，集群设置的文件块大小(blockSize默认为64M)，以及maxSize(mapred.max.split.size)决定。

**splitSize的计算公式：**

```
max{minSize,min{maxSize,blockSize}}，
```

其中minSplitSize大小默认为1B，maxSize为mapred.max.split.size的值。

**总结：**

maxSize变小，则增加map数量



### 2.2 map数的计算方式

单个文件：

文件大小/splitSize>1.1，创建一个split0，文件剩余大小=文件大小-splitSize

.....

剩余文件大小/splitSize<=1.1 将剩余的部分作为一个split

每一个分片对应一个map任务。

总map数：每个文件的map数之和。



### 2.3 调整map数

通过设置参数，来合并map输入的小文件：

```sql
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; 
##执行Map前进行小文件合并

set mapred.max.split.size=128000000; /*每个Map最大输入大小 */
```

注：mapred.map.tasks设置map个数无效。

- 小文件很多，需要减少mapper

```sql
set mapred.max.split.size=100000000;
set mapred.min.split.size.per.node=100000000;
set mapred.min.split.size.per.rack=100000000;
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
前面三个参数确定合并文件块的大小，大于文件块大小128m的，按照128m来分隔，小于128m,大于100m的，
按照100m来分隔，把那些小于100m的（包括小文件和分隔大文件剩下的）合并。
```

​		可以简单的理解为集群对一个表分区下面的文件进行分发到各个节点，之后根据mapred.max.split.size确认要启动多少个map数，逻辑如下  　　

a. 假设有两个文件大小分别为(256M,280M)被分配到节点A，那么会启动两个map，剩余的文件大小为10MB和35MB因为每个大小都不足241MB会先做保留  　

b. 根据参数set mapred.min.split.size.per.node看剩余的大小情况并进行合并,如果值为1，表示a中每个剩余文件都会自己起一个map，这里会起两个，如果设置为大于45*1024*1024则会合并成一个块，并产生一个map  　　

如果mapred.min.split.size.per.node为10*1024*1024，那么在这个节点上一共会有4个map，处理的大小为(245MB,245MB,10MB,10MB，10MB，10MB)，余下9MB  　　

如果mapred.min.split.size.per.node为45*1024*1024，那么会有三个map，处理的大小为(245MB,245MB,45MB)  　　实际中mapred.min.split.size.per.node无法准确地设置成45*1024*1024，会有剩余并保留带下一步进行判断处理  

c. 对b中余出来的文件与其它节点余出来的文件根据mapred.min.split.size.per.rack大小进行判断是否合并，对再次余出来的文件独自产生一个map处理




- 增加mapper数量

```mysql
减少 mapred.max.split.size 的值
```



### 2.4 调整Reducer数

不指定reducer个数的情况下，Hive会猜测确定一个reducer个数，基于以下两个设定：

```properties
参数1：hive.exec.reducers.bytes.per.reducer（每个reduce处理的数据大小，默认为1G）

参数2 ：hive.exec.reducers.max（每个job使用的最大reduce个数，默认为999）
```

计算reducer数的公式：

N=min(参数2，总输入数据量/参数1)

通过设置这两个参数来减少或增加reduce数，案例：

1. 直接设置

```sql
set mapred.reduce.tasks=10; ##设置reduce个数
```
2. 根据数据量计算

```sql
set hive.exec.reducers.bytes.per.reducer=256000000; ##每个reduce处理的数据大小

set hive.exec.reducers.max=2000; ##增加最大reduce个数
```



## 3. Map join优化

### 3.1 大小表关联

一张表十分小，一张表很大，使用mapjoin模式，提交作业的时候先将小表文件放到该作业的DistributedCache中，然后从DistributeCache中取出该小表进行join key / value解释分割放到内存中(可以放大Hash Map等等容器中)。然后扫描大表，看大表中的每条记录的join key /value值是否能够在内存中找到相同join key的记录，如果有则直接输出结果。在适合使用mapjoin的场景，可以在select后使用 /*+ mapjoin(a)*/的方式，或者设置自动开启mapjoin模式参数，set hive.auto.convert.join=true;

例如：
```sql
INSERT OVERWRITE TABLE phone_traffic

SELECT /*+ MAPJOIN(phone_location) */ [l.phone](https://link.zhihu.com/?target=http%3A//l.phone/),p.location,l.traffic from phone_location p join log l on ([p.phone=l.phone](https://link.zhihu.com/?target=http%3A//p.phone%3Dl.phone/))
```



### 3.2 小表过大

如果小表过大，超过mapjoin适合的场景。比如member表100万条记录，日志log表上亿条记录，就不能简单的使用mapjoin了。但通过了解到业务场景，每天活跃的用户数memberid比较少， 则可以先对log表的member_id去重后，使用mapjoin关联member表，然后再和log表通过mapjoin关联。

案例：
```sql

Select * from log a

Left outer join members b	On a.memberid = b.memberid.


优化方法：

Select /*+mapjoin(x)*/* from log a

Left outer join (select /*+mapjoin(c)*/d.*

From (select distinct memberid from log ) c

Join members d

On c.memberid = d.memberid

)x

On a.memberid = b.memberid。
```



## 4. multi insert ,union all

multi insert适合基于同一个源表按照不同逻辑不同粒度处理插入不同表的场景，做到只需要扫描源表一次，job个数不变，减少源表扫描次数。

例如：
```sql

FROM test

INSERT OVERWRITE TABLE count1

SELECT count(DISTINCT test.dqcode)

GROUP BY test.zipcode

INSERT OVERWRITE TABLE count2

SELECT count(DISTINCT test.dqcode)

GROUP BY test.sfcode;
```



union all用好，可减少表的扫描次数，减少job的个数,通常预先按不同逻辑不同条件生成的查询union all后，再统一group by计算,不同表的union all相当于multiple inputs,同一个表的union all,相当map一次输出多条。

例如：

```sql
select country_id,province_id,city_id,type,value

from

(

select country_id,province_id,city_id,'u' type,uuid as value from log where dt='20150101'

union all

select country_id,province_id,city_id,'v' type,visits value from log where dt='20150101'

)t1

group by country_id,province_id,city_id,type,value;
```







## 5. 解决数据倾斜问题

### 5.1 无效数值取值过多

通常的解决思路是，将无效的数据转换为随机数，均匀分配到不同的reduce上处理。

案例：
```sql
Select *

From log a

Join bmw_users b

On a.user_id is not null

And a.user_id = b.user_id

Union all

Select *

from log a

where a.user_id is null.
```

优化方法：
```sql

Select *

from log a

left outer join bmw_users b

on case when a.user_id is null then concat(‘dp_hive’,rand() ) else a.user_id end = b.user_id;
```



### 5.2 有效数值取值过多

如业务需要加工一张订单商品表，订单表order与商品表item通过item_id相关联，基本上每天“热销的商品”（商品被购买次数非常多）是固定的几个item，在未改造前经常出现数据倾斜（这几个热销item_id在对应的reduce上处理时间过长）。

解决思路：先生成一张每天的热销商品清单表（table_item_top)，然后选取top前4的item_id（根据实际情况而定）, 在关联时，若order表的item_id是top前4的item_id，则将item_id转换为item_id_rand的形式，rand的随机取值可能是1~n，随机份数视情况而定；且商品表通过列转行方式，将商品表中的item_id也生成对应的若干行item_id_rand格式数据，然后两表通过item_id_rand关联，热销商品将会随机分配若干份到reduce上并行处理，从而解决倾斜问题。

案例：
```sql
select

t1.order_id,t2.itemname,t2.classname,t2.price

from

(select	order_id,	item_id from order	where dt='20150101'	)t1

left outer join

(select	item_id,	itemname,classname,price from item	where dt='20150101')t2

on t1.item_id= t2.item_id
;


优化方法：

select

t1.order_id,t3.itemname,t3.classname,t3.price

from

(

select

order_id,

item_id,

getRand(item_id,'table_item_top','rand') as item_id_rand

from order

where dt='20150101'

)t1

left outer join

(select

col1 as item_id_rand,

itemname,

classname,

price

from

(

selecr

item_id,

getRand(item_id,'table_item_top','read') as item_id_rand,

itemname,

classname,

price

from item

where dt='20150101'

)t2

lateral view explode(split(item_id_rand,',')) mytable as col1

)t3 on t1.item_id_rand=t3.item_id_rand;
```



### 5.3 不同数据类型关联

一张表日志表，每个商品一条记录，要和商品表关联。但关联却碰到倾斜的问题。日志表中有字符串商品id,也有数字的商品id,类型是string的，但商品中的数字id是bigint的。猜测问题的原因是把s8的商品id转成数字id做hash来分配reduce，所以字符串id的s8日志，都到一个reduce上了。

优化方法：把数字类型转换成字符串类型
```sql

Select * from s8_log a

Left outer join r_auction_auctions b

On a.auction_id = cast(b.auction_id as string);
```



### 5.4 count(distinct )问题

#### 5.4.1 多粒度平级的汇总

比如要计算店铺的uv，还有要计算页面的uv

案例：
```sql

Select shopid,count(distinct uid)

From log group by shopid;

Select pageid, count(distinct uid),

From log group by pageid;
```

由于存在数据倾斜问题，这个结果的运行时间是非常长的。

优化方法：
```sql

Select	type_name,

sum(if(type='page',1,0)) page_uv,

sum(if(type='shop',1,0)) shop_uv

From

(Select type,type_name,uid	From (

Select ‘page’ as type,	Pageid as type_name,	Uid	From log

Union all

Select ‘shop’ as type,	Shopid as type_name,	UidFrom log ) y

Group by type,type_name,uid	) t

group by type_name;
```



#### 5.4.2 多粒度逐层向上的汇总

案例：目前log日志一天有25亿+的数据量，要从日志中按照国家、省份、地市三个粒度分别逐层计算uv及visits 。
```sql

Select country_id,province_id,city_id,count(distinct uuid) uv,count(distinct visits) visits

From log

group by country_id,province_id,city_id;


Select country_id,province_id,count(distinct uuid) uv,count(distinct visits) visits

From log

group by country_id,province_id;


Select country_id,count(distinct uuid) uv,count(distinct visits) visits

From log

group by country_id;
```

优化方法：

按照country、province、city，对uuid与visits打上标签，合并到一起后使用group去重；然后使用row_number函数，统计不同粒度的排名，最终产生临时表tmp。
```sql

insert overwrite table tmp1 partition(dt='20150101')

select

country_id,province_id,city_id,type,

row_number() over(partition by country_id,province_id,city_id,type order by value) as city_rn,

row_number() over(partition by country_id,province_id,type order by value) as province_rn,

row_number() over(partition by country_id,type order by value) as country_rn

from

(select country_id,province_id,city_id,type,value

from

(

select country_id,province_id,city_id,'u' type,uuid as value from log where dt='20150101'

union all

select country_id,province_id,city_id,'v' type,visits value from log where dt='20150101'

)t1

group by country_id,province_id,city_id,type,value

)t2;


最后，按照三个层级粒度分别对uv和visits进行汇总统计。

##sql1:

select

country_id,province_id,city_id,

sum(case when type='u' and city_rn=1 then 1 else 0 end) as city_uv,

sum(case when type='v' and city_rn=1 then 1 else 0 end) as city_visits

from	tmp1	where dt='20150101'	group by	country_id,province_id,city_id;


##sql2:

select	country_id,province_id,

sum(case when type='u' and province_rn=1 then 1 else 0 end) as province_uv,

sum(case when type='v' and province_rn=1 then 1 else 0 end) as province_visits

from	tmp1	where dt='20150101'	group by	country_id,province_id;


##sql3:

select

country_id,province_id,

sum(case when type='u' and country_rn=1 then 1 else 0 end) as country_uv,

sum(case when type='v' and country_rn=1 then 1 else 0 end) as country_visits

from	tmp1	where dt='20150101'	group by	country_id,province_id;
```



## 6. 尽量减少job数

hive对union all优化只局限于非嵌套查询。

案例：

```sql
select * from

(select * from t1	Group by c1,c2,c3

Union all

Select * from t2	Group by c1,c2,c3) t3

Group by c1,c2,c3;
```

从业务逻辑上说，子查询内的group by 怎么都看显得多余（功能上的多余,除非有count(distinct)），如果不是因为hive bug或者性能上的考量(曾经出现如果不子查询group by ，数据得不到正确的结果的hive bug)。所以这个hive按经验转换成

```sql
select * from

(select * from t1 Union all	Select * from t2) t3

Group by c1,c2,c3;
```

经过测试，并未出现union all的hive bug,数据是一致的。mr的作业数有3减少到1



## 7. sql书写规范

尽量尽早地过滤数据，减少每个阶段的数据量,对于分区表要加分区。

案例：

```sql
SELECT a.key,col1,col2,col3,col4

FROM

A a LEFT OUTER JOIN B b

ON a.key = b.key and a.dt=‘20150101’ and b.dt=‘20150101';
```


优化方法1：

```sql
SELECT ...	FROM A

LEFT OUTER JOIN B ON A.key = B.key

WHERE A.dt='20150101'	AND B.dt='20150101';
```


优化方法2：

```sql
SELECT a.key,col1,col2,col3,col4	FROM

(SELECT key,col1,col2	FROM A	WHERE dt='20150101') a

LEFT OUTER JOIN

(SELECT key,col3,clo4	FROM B	WHERE dt='20150101' ) b 

ON a.key = b.key;
```







#3.优化-运行方面

##1. 并发执行
```sql
hive.exec.parallel=true; /*默认为false */

hive.exec.parallel.thread.number=8; 
```



## 2. 合并小文件

```sql
hive.merge.mapfiles=true ：合并map输出

hive.merge.mapredfiles=false ：合并reduce输出

hive.merge.size.per.task=256*1000*1000 ：合并文件的大小

hive.mergejob.maponly=true：如果支持CombineHiveInputFormat则生成只有Map的任务执行merge

hive.merge.smallfiles.avgsize=16000000：文件的平均大小小于该值时，会启动一个MR任务执行merge。
```



## 3. map聚合

```sql
hive.map.aggr=true; 在map中会做部分聚集操作，效率更高但需要更多的内存。

hive.groupby.mapaggr.checkinterval：在Map端进行聚合操作的条目数目
```



## 4. 负载均衡(skewindata)

hive.groupby.skewindata=true：数据倾斜时负载均衡，当选项设定为true，生成的查询计划会有两个MRJob。第一个MRJob 中，

Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的GroupBy Key

有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MRJob再根据预处理的数据结果按照GroupBy Key分布到

Reduce中（这个过程可以保证相同的GroupBy Key被分布到同一个Reduce中），最后完成最终的聚合操作。



## 5. 本地模式(小任务)

需要满足以下条件：

　　1. job的输入数据大小必须小于参数：hive.exec.mode.local.auto.inputbytes.max(默认128MB)

　　2. job的map数必须小于参数：hive.exec.mode.local.auto.tasks.max(默认4)

　　3. job的reduce数必须为0或者1

```sql
hive.exec.mode.local.auto.inputbytes.max=134217728

hive.exec.mode.local.auto.tasks.max=4

hive.exec.mode.local.auto=true

hive.mapred.local.mem：本地模式启动的JVM内存大小
```



## 6. Strict Mode

hive.mapred.mode=true，严格模式不允许执行以下查询：

- 分区表上没有指定了分区

- 没有limit限制的order by语句

- 笛卡尔积：JOIN时没有ON语句



## 7 使用索引

```sql
hive.optimize.index.filter：自动使用索引

hive.optimize.index.groupby：使用聚合索引优化GROUP BY操作
```





参考：

set split size  https://www.jianshu.com/p/0ecccfc38923 