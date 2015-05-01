# Spark入门

## Intro
当下互联网界最火的是什么？毫无疑问是大数据！当然其中不乏泡沫之嫌，但是大数据的红火可以说是互联网发展的必然。  

对笔者来说，大数据的魅力在于它所带来的挑战和机遇。  
对于整个行业（包括传统行业）来说，如何利用手头的大数据，如何高效存储、快速分析与有效挖掘都是一个需要持续改善的过程，这是整个行业的挑战；而对于我们IT从业者来说，如何快速跟上技术发展的脚步，如何从茫茫技术中选择适合自己适合公司的工具，又如何在繁多又繁琐的数据中挖到金矿，都是我们这些“键盘侠”的挑战。而这些挑战同时也意味着机遇。我们可以利用大数据帮助行业快速正确地发展，另一个很实际的问题是，我们可以通过大数据获取更加丰厚的回报（这也是关乎大家切身利益的）。  

当我们谈大数据的时候，我们都在谈什么？大数据无外乎这几点：存储，分析、挖掘；当下大数据的主流架构就是我们熟知的Hadoop。Hadoop的核心是MapReduce和HDFS，他们在过去较长一段时间都处于统治地位，而MR的劣势（不灵活、中间结果保存在HDFS上导致和HDFS过多交互使得处理较慢等等）开始被很多开发者诟病；这时候，Spark站出来了！对于很多人来说，Spark仿佛是一夜之间冒出来然后开始风靡整个业界。Apache也将Spark放在很重要的战略地位。  

关于Spark，网上的资料太多太多，在这里，笔者只想从Spark源码的角度去剖析Spark。一方面作为自己学习源码的记录，另一方面也希望能和大家交流学习Spark的心得。  

## Spark简介
Spark是什么？它最早是2009年伯克利大学AMP实验室开发的一个开源大数据计算处理框架，2010年成为Apache的开源项目。相较于Hadoop的MR，基于内存计算的Spark比MR快10到100倍。它的活跃开发者已经超过了MapReduce。Spark是使用Scala语言开发的，但是它支持Scala、Java、Python、Clojure以及R在内的多种语言。我们可以选择我们熟悉的编程语言进行Spark应用开发。  

首先，我们需要对Spark有一个整体的把握，下面这张图是Spark的主要成员：  

![Spark-stack](http://spark.apache.org/images/spark-stack.png)

我们可以看到Spark除了基础计算框架，还包括了SparkSQL（取代了Shark的位置）、流处理、机器学习以及图计算，非常全面。这也是Spark的优势所在。  
**SparkSQL**：Spark数据集的类SQL查询。当前Spark SQL还处于alpha阶段，一些API在将将来的版本中可能会有所改变；  
**Spark Steaming**：处理实时的数据流，基于微批量的方式计算和处理；  
**Spark MLlib**：Spark的机器学习库，它包含通用的学习算法和工具；  
**Spark GraphX**：GraphX是Spark针对图计算和并行图计算开发的API；  

Spark的主要特性有：  
**高效性**：这是Spark引以为傲的地方，它的[官网](https://spark.apache.org/)第一条就是这个，大家感受下：Run programs up to 100x faster than Hadoop MapReduce in memory, or 10x faster on disk. 字里行间透露的是Spark的骄傲！  
**易用性**：Spark支持Java、Python、Scala等编程语言，你还可以使用Shell进行交互；而且它还提供了80多种算子使得我们操作起来更加方便灵活；  
**通用性**：亦即我们上文提到的它包含图计算、机器学习以及流处理，SQL查询；  
**移植性**：Spark支持三种模式，Standalone、Yarn以及Mesos，它可以访问的数据源包括HDFS，HBase，Cassandra等等；  

## Spark源码结构
笔者想通过阅读Spark源码来深入理解Spark运行的原理与机制；了解源码的结构就是我们的第一步。我们选取的Spark版本是1.3.0。  
我们先介绍一些主要的包：  
**core**：这是我们最关注的包，Spark核心的API代码都在里面；  
**example**：Spark使用的示例；   
**sql**：SparkSQL实现的相关代码；  
**graphx**：Spark图计算相关实现代码；  
**yarn**：支持Spark在Yarn上运行的相关实现代码；  
**streaming**：SparkStreaming，Spark流处理相关实现代码；  
**repl**：Spark实现shell交互的相关代码；  
**mllib**：Spark机器学习相关实现代码；  

我们在接下来的时间里会比较深入了解这些包里的内容，当然这会是漫长但是充满惊喜的旅程；

**我们已经在路上了！**