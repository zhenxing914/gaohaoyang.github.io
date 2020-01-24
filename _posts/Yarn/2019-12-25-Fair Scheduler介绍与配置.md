在一个公司内部的Hadoop Yarn集群，肯定会被多个业务、多个用户同时使用，共享Yarn的资源，如果不做资源的管理与规划，那么整个Yarn的资源很容易被某一个用户提交的Application占满，其它任务只能等待，这种当然很不合理，我们希望每个业务都有属于自己的特定资源来运行MapReduce任务，Hadoop中提供的公平调度器–Fair Scheduler，就可以满足这种需求。

Fair Scheduler将整个Yarn的可用资源划分成多个资源池，每个资源池中可以配置最小和最大的可用资源（内存和CPU）、最大可同时运行Application数量、权重、以及可以提交和管理Application的用户等。

## 1.根据用户名分配资源池

![fair scheduler](https://tva1.sinaimg.cn/large/006tNbRwly1ga8xkbo7duj30p90astd5.jpg)

如图所示，假设整个Yarn集群的可用资源为100vCPU，100GB内存，现在为3个业务各自规划一个资源池，另外，规划一个default资源池，用于运行其他用户和业务提交的任务。如果没有在任务中指定资源池（通过参数mapreduce.job.queuename），那么可以配置使用用户名作为资源池名称来提交任务，即用户businessA提交的任务被分配到资源池businessA中，用户businessC提交的任务被分配到资源池businessC中。除了配置的固定用户，其他用户提交的任务将会被分配到资源池default中。

这里的用户名，就是提交Application所使用的Linux/Unix用户名。

另外，每个资源池可以配置允许提交任务的用户名，比如，在资源池businessA中配置了允许用户businessA和用户lxw1234提交任务，如果使用用户lxw1234提交任务，并且在任务中指定了资源池为businessA，那么也可以正常提交到资源池businessA中。

## 2.根据权重获得额外的空闲资源

在每个资源池的配置项中，有个weight属性（默认为1），标记了资源池的权重，当资源池中有任务等待，并且集群中有空闲资源时候，每个资源池可以根据权重获得不同比例的集群空闲资源。

比如，资源池businessA和businessB的权重分别为2和1，这两个资源池中的资源都已经跑满了，并且还有任务在排队，此时集群中有30个Container的空闲资源，那么，businessA将会额外获得20个Container的资源，businessB会额外获得10个Container的资源。

 

## 3.最小资源保证

在每个资源池中，允许配置该资源池的最小资源，这是为了防止把空闲资源共享出去还未回收的时候，该资源池有任务需要运行时候的资源保证。

比如，资源池businessA中配置了最小资源为（5vCPU，5GB），那么即使没有任务运行，Yarn也会为资源池businessA预留出最小资源，一旦有任务需要运行，而集群中已经没有其他空闲资源的时候，这个最小资源也可以保证资源池businessA中的任务可以先运行起来，随后再从集群中获取资源。

 

## 4.动态更新资源配额

Fair Scheduler除了需要在yarn-site.xml文件中启用和配置之外，还需要一个XML文件来配置资源池以及配额，而该XML中每个资源池的配额可以动态更新，之后使用命令：yarn rmadmin –refreshQueues 来使得其生效即可，不用重启Yarn集群。

需要注意的是：动态更新只支持修改资源池配额，如果是新增或减少资源池，则需要重启Yarn集群。

## 5.Fair Scheduler配置示例

以上面图中所示的业务场景为例。

### 5.1 yarn-site.xml中的配置：

```xml
<!– scheduler start –>
<property>
<name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
</property>
<property>
<name>yarn.scheduler.fair.allocation.file</name>
<value>/etc/hadoop/conf/fair-scheduler.xml</value>
</property>
<property>
<name>yarn.scheduler.fair.preemption</name>
<value>true</value>
</property>
<property>
<name>yarn.scheduler.fair.user-as-default-queue</name>
<value>true</value>
<description>default is True</description>
</property>
<property>
<name>yarn.scheduler.fair.allow-undeclared-pools</name>
<value>false</value>
<description>default is True</description>
</property>
<!– scheduler end –>
```

- **yarn.resourcemanager.scheduler.class**

  配置Yarn使用的调度器插件类名；

  Fair Scheduler对应的是：

  org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler

- **yarn.scheduler.fair.allocation.file**

  配置资源池以及其属性配额的XML文件路径（本地路径）；

- **yarn.scheduler.fair.preemption**

  开启资源抢占。

- **yarn.scheduler.fair.user-as-default-queue**

  设置成true，当任务中未指定资源池的时候，将以用户名作为资源池名。这个配置就实现了根据用户名自动分配资源池。

- **yarn.scheduler.fair.allow-undeclared-pools**

  是否允许创建未定义的资源池。

  如果设置成true，yarn将会自动创建任务中指定的未定义过的资源池。设置成false之后，任务中指定的未定义的资源池将无效，该任务会被分配到default资源池中。

 

### 5.2 fair-scheduler.xml中的配置:

```xml
<?xml version=”1.0″?>
<allocations>
<!– users max running apps –>
<userMaxAppsDefault>30</userMaxAppsDefault>
<!– queues –>
<queue name=”root”>
<minResources>51200mb,50vcores</minResources>
<maxResources>102400mb,100vcores</maxResources>
<maxRunningApps>100</maxRunningApps>
<weight>1.0</weight>
<schedulingMode>fair</schedulingMode>
<aclSubmitApps> </aclSubmitApps>
<aclAdministerApps> </aclAdministerApps>

<queue name=”default”>
<minResources>10240mb,10vcores</minResources>
<maxResources>30720mb,30vcores</maxResources>
<maxRunningApps>100</maxRunningApps>
<schedulingMode>fair</schedulingMode>
<weight>1.0</weight>
<aclSubmitApps>*</aclSubmitApps>
</queue>

<queue name=”businessA”>
<minResources>5120mb,5vcores</minResources>
<maxResources>20480mb,20vcores</maxResources>
<maxRunningApps>100</maxRunningApps>
<schedulingMode>fair</schedulingMode>
<weight>2.0</weight>
<aclSubmitApps>businessA,lxw1234 group_businessA,group_lxw1234</aclSubmitApps>
<aclAdministerApps>businessA,hadoop group_businessA,supergroup</aclAdministerApps>
</queue>

<queue name=”businessB”>
<minResources>5120mb,5vcores</minResources>
<maxResources>20480mb,20vcores</maxResources>
<maxRunningApps>100</maxRunningApps>
<schedulingMode>fair</schedulingMode>
<weight>1</weight>
<aclSubmitApps>businessB group_businessA</aclSubmitApps>
<aclAdministerApps>businessA,hadoop group_businessA,supergroup</aclAdministerApps>
</queue>

<queue name=”businessC”>
<minResources>5120mb,5vcores</minResources>
<maxResources>20480mb,20vcores</maxResources>
<maxRunningApps>100</maxRunningApps>
<schedulingMode>fair</schedulingMode>
<weight>1.5</weight>
<aclSubmitApps>businessC group_businessC</aclSubmitApps>
<aclAdministerApps>businessC,hadoop group_businessC,supergroup</aclAdministerApps>
</queue>
</queue>
</allocations>
```

- **minResources**

  最小资源

- **maxResources**

  最大资源

- **maxRunningApps**

  最大同时运行application数量

- **weight**

  资源池权重

- **aclSubmitApps**  acl = access control list

  允许提交任务的用户名和组；

  格式为： 用户名 用户组

  当有多个用户时候，格式为：用户名1,用户名2 用户名1所属组,用户名2所属组

- **aclAdministerApps**

  允许管理任务的用户名和组；

  格式同上。

​    Fair Scheduer各资源池配置及使用情况，在ResourceManager的WEB监控页面上也可以看到：

![fair scheduler](https://tva1.sinaimg.cn/large/006tNbRwly1ga8xkez73qj30xj0de11n.jpg)



参考： 

http://lxw1234.com/archives/2015/10/536.htm

