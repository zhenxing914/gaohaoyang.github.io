



> **Hive任务申请资源流程**
>
> 执行一个hive语句，首先会启动一个AM进程。执行map任务时，根据map数启动相同个数container。map执行完成后，根据reducer个数，启动相同个数的container。



## 1. Container资源设置

**以下参数是在用户自己的mr应用程序中配置就可以生效（mapred-default.xml）**

参数介绍

```yml
mapreduce.map.memory.mb    #默认1024M，一个map启动的container是1024M

mapreduce.map.cpu.vcores   #默认1个vcore，一个map任务默认使用一个虚拟核运行

mapreduce.reduce.memory.mb  #默认3072M，一个map启动的container是3072

mapreduce.reduce.cpu.vcores #默认1个vcore，一个reduce任务默认使用一个虚拟核运行。
```



调优就是尽可能的让集群资源充分利用，这里需要根据具体的需求和集群资源情况来定。
例如不考虑内存溢出等情况, 集群资源如下

| Memory Total | VCores Total |
| ------------ | ------------ |
| 320G         | 80           |

如果数据比较均匀，应该尽可能的设置成如下：

| mapreduce.map.memory.mb | mapreduce.reduce.memory.mb | mapreduce.map.cpu.vcores | mapreduce.reduce.cpu.vcores |
| ----------------------- | -------------------------- | ------------------------ | --------------------------- |
| 4096                    | 4096                       | 1                        | 1                           |

这样并发数能到

```
max(320G/4G，80vcore/1vcore)=80
```

上面是map核reduce都到了最大的80的并发，集群利用最充分

一般来说，我们默认mapreduce.map.cpu.vcores和mapreduce.reduce.cpu.vcores为1个就好了，但是对于一个map和一个reduce的container的内存大小设置成了4G，如果一个map和一个reduce处理的任务很小，那又会很浪费资源，这时，对于map来说，可以用前面说的小文件整合，设置mapreduce.input.fileinputformat.split来解决map的大小，尽可能接近4G，但是又要注意可能出现的内存溢出的情况。

对于reduce，1个container启动用了4G内存，那这4G内存也应尽可能的充分使用，这时候，我们尽量的评估进入到reduce的数据大小有多少，合理的设置reduceTask数，这一步是比较麻烦的，因为这里如果出现数据倾斜将会导致oom内存溢出错误。

前面说到了，并发数受到 集群总内存/container的限制，同时，并发数也会受到集群vcore的限制，还是上面那个例子，例如集群资源为320G，80vcore，我一个map任务为2G，由于受到cpu的限制，最多同时80个vcore的限制，那么内存使用只能使用160G。这显然是浪费资源了。

对于mapreduce.map.cpu.vcores和mapreduce.reduce.cpu.vcores，为什么默认是1呢，在集群的内存/cpu很小的情况下，能否一个map端将这两个值设置成2或者更大呢。这是当然可以的，但是，即使我们将这个设置成2，任务的并发并不会是 Vcores Total/2的关系，并发仍然将是上面两条决定的。举个例子，还是320G，80vcore集群。我们设置mapreduce.map.memory.mb为4G，mapreduce.map.cpu.vcores为2， 很多人以为我一个map需要两个核，那么80vcore/2vcore=40，那么我们并发最大只能用到40*4=160G的内存，其实这是错误的，这种情况，我们任然基本上能将内存占满，并发数仍然能到80个。这个时候， mapreduce.map.cpu.vcores基本就失效了。后来仔细想了想，一个map或者reduce任务，里面的数据应该并不可能会有多线程并发，但是mapreduce.map.cpu.vcores为什么会有这个参数呢，后来想了一下，一个map或者reduce任务，代码的执行肯定是一个线程，但是任务的状态等监控，垃圾回收等，是可以使用另外一个线程来运行的，这也是将mapreduce.map.cpu.vcores设置成2可能会快一点的效果。

我曾经碰到一个cpu十分充足的集群，vcore和内存比例是1比1，但是为了让数据不倾斜，我们的mapreduce.reduce.memory.mb至少要到4G，那么这时候，其实cpu就只能利用1/4了，这时候cpu很充足，我便尝试将mapreduce.map.cpu.vcores设置成2.其实这样也并不是说我一定每个map都能使用到2个vcore，只不过有时候，有的任务状态监控，jvm垃圾回收等，就有了另外一个vcore来运行了。

mapreduce.map.cpu.vcores， 这个参数貌似在公平队列是没用的，vCores用于较大的群集，以限制不同用户或应用程序的CPU。如果您自己使用YARN，则没有理由限制容器CPU。这就是为什么在Hadoop中默认甚至不考虑vCore的原因，capacity-schedule调度下才有用。



## 2. Yarn资源配置

**应该在yarn启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）**

```
yarn.scheduler.minimum-allocation-vcores 

yarn.scheduler.maximum-allocation-vcores
```

参数解释：单个任务可申请的最小/最大虚拟CPU个数。比如设置为1和4，则运行MapRedce作业时，每个Task最少可申请1个虚拟CPU，最多可申请4个虚拟CPU。



原文链接：https://blog.csdn.net/mlljava1111/article/details/51867883
原文链接：https://blog.csdn.net/T1DMzks/article/details/80204420



