# Spark基本概念

## Intro
我们马上就要开始深入Spark的源码了，在阅读源码之前，我们有必要了解一些Spark中的常用概念，这会有助于我们后面的学习。

## Definition
我们在这里只是简单的介绍一下我们会经常遇到的一些关键术语，我们会在后面分别对它们进行详细的解读。  

### 1.SparkContext
Spark的入口。它负责连接Spark集群，创建RDD以及累积量、广播量。本质上来说，SparkContext是Spark的对外接口，负责集群对外交互。要想理解Spark，我们就必须了解SparkContext。我们在创建SparkContext对象的时候，sparkcontext内部就会创建Task Scheduler和DAG Scheduler。  
**DAG Scheduler**：面向Stage的调度层，为Job生成以Stage组成的DAG，提交TaskSet给Task Scheduler执行。   
**Task Scheduler**：DAG Scheduler以Stage为单位，提Task给Task Scheduer，TaskScheduler做接收Task、接收分到的资源和Executor、维护信息、与backend打交道、把任务分配好等事情。

### 2.RDD
弹性分布式数据集，Spark的核心。Spark最大的亮点就是提出了RDD这一概念，它是一个并行的、容错的数据结构，只读而且可以恢复，RDD本身不做物理存储，而是通过保存足够的信息然后在实际的存储中计算RDD。

### 3.Operations
这里是指RDD上的操作，主要有两种：Transformation，Action。Transformation从当前的RDD创建一个新的RDD，而Action在RDD上进行计算，然后将结果返回给驱动程序。无论执行了多少次Transformation操作，RDD都不会真正执行运算（延迟计算），只有当Action操作被执行时，运算才会触发。

### 4.Application
基于Spark的用户程序，包含了一个Driver program和集群中多个的Executor。  
**Driver program**：运行Application的main()函数并且创建SparkContext，通常用SparkContext代表Driver Program；  
**Executor**：为Application运行在Worker上的一个进程，该进程负责运行Task，并且负责将数据存在内存或磁盘上。每个Application都有各自独立的Executors。  
**Job**：Spark作业。每个Job包含一系列RDD，以及这些RDD上的各种Operation；  
**Task**：被送到executor上执行的工作单元；  
**Stage**：每个Job会被拆分很多组Task，每组任务被称为Stage，也可称TaskSet；  

## Let's Start
看完了上面那些抽象的概念，我们在这里将要小试牛刀，一起找一下阅读源码的感觉。  

我们看到spark源码的根目录下，有一个README.md文档，这个文档我们基本上在所有的开源项目里都能看到，里面是整个project的大致介绍。首先，我们看到的是Spark的官方定义：  

`Spark is a fast and general cluster computing system for Big Data. It provides`  
`high-level APIs in Scala, Java, and Python, and an optimized engine that`  
`supports general computation graphs for data analysis. It also supports a`  
`rich set of higher-level tools including Spark SQL for SQL and structured`  
`data processing, MLlib for machine learning, GraphX for graph processing,`  
`and Spark Streaming for stream processing.`

我们可以看到Spark官方非常认可Spark的fast和general两大特性。这里面只提到了Scala、Python以及Java的API，实际上，Spark已经开始和R走到一块。  
我们在文档接近末尾处发现这样一段：  

`A Note About Hadoop Versions`  

`Spark uses the Hadoop core library to talk to HDFS and other Hadoop-supported`  
`storage systems. Because the protocols have changed in different versions of`  
`Hadoop, you must build Spark against the same version that your cluster runs.`  

这一段很重要，提到了Spark对Hadoop的依赖，Spark要想访问HDFS或者其他Hadoop支持的文件系统（如本地文件、S3、FTP等等），都必须通过Hadoop的核心库进行交互。而且我们集群上的Hadoop版本必须一致。  

你看，这个README.md文件就非常有价值，让我们看到了一些平时在别人博客或者论坛看到的知识点，而我们知道了它们的出处，是不是也小有成就感。接下来我们就真的要开始源代码之旅了，希望我们都能收获满满。



**参考**：  
[理解Spark的核心RDD](http://www.infoq.com/cn/articles/spark-core-rdd)  
[RDD：基于内存的集群计算容错抽象](http://shiyanjun.cn/archives/744.html)  
[RDD的原理](https://github.com/jackfengji/test_pro/wiki/RDD%E7%9A%84%E5%8E%9F%E7%90%86)  
