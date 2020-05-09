

自动驾驶的平台需要云计算，比如大量的机器学习和深度学习训练，高清地图，模拟仿真模块，还有车联网。

这里看到一篇 Spark 和 Flink 的比较文章，借机转载，以后要重新学习这个领域的新东西。

[Introduction to Apache Flink for Spark Developers : Flink vs Sparkblog.madhukaraphatak.com](https://link.zhihu.com/?target=http%3A//blog.madhukaraphatak.com/introduction-to-flink-for-spark-developers-flink-vs-spark/)

Apache Flink 是新一代通用大数据处理引擎，旨在统一不同的数据负载。 听起来像 Apache Spark 吗？ 是的。 Flink 正试图解决 Spark 试图解决的同样问题。 这两个系统都旨在构建单一平台，可以在其中运行批处理，流媒体，交互式，图形处理，机器学习等。因此，Flink 与 Spark 的意识形态中介没有太大差别。 但它们在实施细节方面确实存在很大差异。

下面比较 Spark 和 Flink 的不同。 一些方法在两个框架中都是相同的，而有些方法有很大不同。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gelywj3981j30lg0dp75w.jpg" alt="Apache 两个开源项目比较：Flink vs Spark" style="zoom:80%;" />



## 1. 抽象

在 Spark 中，对于批处理，有 RDD 抽象和 DStream 用于流式传输，这是内部 RDD 本身。因此，在 Spark 下面代表的所有数据都使用 RDD 抽象来表示。

在 Flink 中，为批处理数据集提供了数据集抽象，为流应用程序提供了 DataStream。它们听起来与 RDD 和 DStreams 非常相似，但它们不是。

**差异是以下几点：**

### 1. 数据集在运行时只是计划

在 Spark 中，RDD 在运行时表示为 java 对象。随着 project Tungsten 的推出，它有点变化。但在 Apache Flink 中，数据集被表示为一个逻辑计划。这听起来很熟悉吗？是的，它们就像 Spark 中的 Dataframe。所以在 Flink 中可以像使用优化器优化的一等公民那样获得像 api 这样的 Dataframe。但是在 Spark RDD 之间不做任何优化。

Flink 的数据集就像 Spark 的 Dataframe API，在执行之前进行了优化。

在 Spark 1.6 中，数据集 API 被添加到 spark 中，这可能最终取代 RDD 抽象。

### 2. Dataset 和 DataStream 是独立的 API

在 Spark 中，所有不同的抽象，如 DStream，Dataframe 都建立在 RDD 抽象之上。但在 Flink 中，Dataset 和 DataStream 是基于顶级通用引擎构建的两个独立抽象。虽然它们模仿了类似的 API，但是在 DStream 和 RDD 的情况下，无法将它们组合在一起。尽管在这方面有一些努力，但最终结果还不够明确。

不能将 DataSet 和 DataStream 组合在一起，如 RDD 和 DStreams。

因此，虽然 Flink 和 Spark 都有类似的抽象，但它们的实现方式不同。



## 2. 内存管理

直到 Spark 1.5，Spark 使用 Java 堆来缓存数据。虽然项目开始时更容易，但它导致了内存不足（OOM）问题和垃圾收集（gc）暂停。因此，从 1.5 开始，Spark 进入定制内存管理，称为 project tungsten。

Flink 从第一天起就开始定制内存管理。实际上，这是 Spark 向这个方向发展的灵感之一。不仅 Flink 将数据存储在它的自定义二进制布局中，它确实直接对二进制数据进行操作。在 Spark 中，所有数据帧操作都直接在 Spark 1.5 的 project tungsten 二进制数据上运行。

在 JVM 上执行自定义内存管理可以提高性能并提高资源利用率。



## 3. 实施语言

Spark 在 Scala 中实现。它提供其他语言的 API，如 Java，Python 和 R。

Flink 是用 Java 实现的。它确实提供了 Scala API。

因此，与 Flink 相比，Spark 中的选择语言更好。在 Flink 的一些 scala API 中，java 抽象也是 API 的。这会有所改进，因为已经使 scala API 获得了更多用户。



## 4. API

Spark 和 Flink 都模仿 scala 集合 API。所以从表面来看，两者的 API 看起来非常相似。
![Apache 两个开源项目比较：Flink vs Spark](https://tva1.sinaimg.cn/large/007S8ZIlgy1gelyzyteggj30j60gdtax.jpg)



## 5. 流

Apache Spark 将流式处理视为快速批处理。 Apache Flink 将批处理视为流处理的特殊情况。这两种方法都具有令人着迷的含义。

两种不同方法的差异或含义:

### 1. 实时与近实时

Apache Flink 提供事件级处理，也称为实时流。它与 Storm 模型非常相似。

Spark 只有不提供事件级粒度的最小批处理（mini-batch）。这种方法被称为近实时。

Spark 流式处理是更快的批处理，Flink 批处理是有限的流处理。

虽然大多数应用程序都可以近乎实时地使用，但很少有应用程序需要事件级实时处理。这些应用程序通常是 Storm 流而不是 Spark 流。对于他们来说，Flink 将成为非常有趣的选择。

### 2. 能够将历史数据 / 流相结合

运行流处理作为更快批处理的优点之一是，我们可以在两种情况下使用相同的抽象。 Spark 非常支持组合批处理和流数据，因为它们都使用 RDD 抽象。

在 Flink 的情况下，批处理和流式传输不共享相同的 API 抽象。因此，尽管有一些方法可以将基于历史文件的数据与流相结合，但它并不像 Spark 那样干净。

在许多应用中，这种能力非常重要。在这些应用程序中，Spark 代替 Flink 流式传输。

### 3. 灵活的窗口

由于最小批处理的性质，Spark 现在对窗口的支持非常有限。允许根据处理时间窗口批量处理。
与其他任何系统相比，Flink 提供了非常灵活的窗口系统。 Window 是 Flink 流 API 的主要焦点之一。它允许基于处理时间、数据时间和无记录等的窗口。这种灵活性使 Flink 流 API 与 Spark 相比非常强大。



## 6. SQL 界面

截至目前，最活跃的 Spark 库之一是 spark-sql。 Spark 提供了像 Hive 一样的查询语言和像 DSL 这样的 Dataframe 来查询结构化数据。它是成熟的 API 并且在批处理中广泛使用并且很快将在流媒体世界中使用。

截至目前，Flink Table API 仅支持 DSL 等数据帧，并且仍处于测试阶段。有计划添加 sql 接口，但不确定何时会落在框架中。

目前为止，Spark 与 Flink 相比有着不错的 SQL 故事。



## 7. 数据源集成

Spark 数据源 API 是框架中最好的 API 之一。数据源 API 使得所有智能资源如 NoSQL 数据库，镶木地板，优化行列（Optimized Row Columnar，ORC）成为 Spark 上的头等公民。此 API 还提供了在源级执行谓词下推（predicate push down）等高级操作的功能。

Flink 仍然在很大程度上依赖于 map / reduce InputFormat 来进行数据源集成。虽然它是足够好的提取数据 API，但它不能巧妙地利用源能力。因此 Flink 目前落后于目前的数据源集成技术。



## 8. 迭代处理

Spark 最受关注的功能之一就是能够有效地进行机器学习。在内存缓存和其他实现细节中，它是实现机器学习算法的真正强大的平台。

虽然 ML 算法是循环数据流，但它表示为 Spark 内部的直接非循环图。通常，没有分布式处理系统鼓励循环数据流，因为它们变得难以理解。

但是 Flink 对其他人采取了一些不同的方法。它们在运行时支持受控循环依赖图（cyclic dependence graph）。这使得它们与 DAG 表示相比以非常有效的方式表示 ML 算法。因此，Flink 支持本机平台中的迭代，与 DAG 方法相比，可实现卓越的可扩展性和性能。



## 9. 流作为平台与批处理作为平台

Apache Spark 来自 Map / Reduce 时代，它将整个计算表示为数据作为文件集合的移动。这些文件可能作为磁盘上的阵列或物理文件驻留在内存中。这具有非常好的属性，如容错等。
但是 Flink 是一种新型系统，它将整个计算表示为流处理，其中数据有争议地移动而没有任何障碍。这个想法与像 akka-streams 这样的新的反应流系统非常相似。



## 10. 成熟

Flink 像批处理这样的部分已经投入生产，但其他部分如流媒体，Table API 仍在不断发展。这并不是说在生产中就没人使用 Flink 流。
<img src="https://static001.infoq.cn/resource/image/d3/75/d348b4b1562dcf61d97a1e64ae9c0d75.jpg" alt="Apache 两个开源项目比较：Flink vs Spark" style="zoom:67%;" />



## 11. 总结

​		与 Flink 相比，Spark 是一个非常成熟和完整的框架，但 Flink 确实带来了非常有趣的想法，如自定义内存管理，数据集 API 等。 Spark 社区正在认识它并将这些想法融入到 Spark 中。 所以从这个意义上来说，Flink 正在将大数据处理完全提升到下一个层次。



**本文来源**
https://zhuanlan.zhihu.com/p/68206953