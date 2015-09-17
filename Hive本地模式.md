# Hive 本地模式
__author:leotse__

### Local Mode
我们都知道，Hive是通过将HiveQL转化为MR job进行数据查询和处理。当然，不是全部的HQL语句都需要转为MR，比如我们常见的：  
`SELECT * FROM access_log;`  
这时候，Hive Shell能很快地返回给我们结果，这是因为这里用到的是Hive的**本地模式**。  
Hive查询耗时太长，一直都是大家吐槽的对象，在数据量大的情况下，当然微不足道，但是当我们的查询量很小亦即查询数据很少的时候，Hive启动的耗时将会显得非常刺眼，因此在0.7版本以后，Hive开始支持本地模式。

除了上面的示例使用到了本地模式，如果WHERE语句中只是分区字段这种情况，也是无需使用MR job的：  
`SELECT * FROM access_log`  
`WHERE dt='20150916'`  
`LIMIT 10;`  
这是因为分区字段在HDFS中保存实际上是目录结构，这些都是不用通过计算获取的（包含LIMIT）。

除此之外，我们还可以怎么利用本地模式帮助我们快速处理少量数据呢？我们可以通过设置下面变量打开本地模式：  
`set hive.exec.mode.local.auto=true;`  
默认情况下这个设置为false，也就是说，Hive使用MR来执行其他的所有操作（区别上面介绍的两种）。

我们来感受下，下面截图为没有使用本地模式时查询数据条目数：  
![hive_local_mode_false](https://github.com/leotse90/SparkNotes/blob/master/images/hive_local_mode_false.png)  
我们看到这个简单的操作用了34.321s，一共也就192条数据，这个固然和机器有关，但是MR启动以及计算导致时间成本太高。我们在下面的示例里打开了本地模式：  
![hive_local_mode_true](https://github.com/leotse90/SparkNotes/blob/master/images/hive_local_mode_true.png)  
1.322s!，我们看到耗时明显降低，这就是本地模式的威力。

当然，使用本地模式也有一些条件：  
1.**输入数据的size**：我们用参数hive.exec.mode.local.auto.inputbytes.max来指定本地模式处理的最大输入数据，默认为128MB；  
2.**Mapper的数量**：参数hive.exec.mode.local.auto.tasks.max指定了本地模式下Mapper的最大数量，默认为4；  
3.**Reducer的数量**：Reducer数量必须是0或1；

我们在使用Hive时，最好在$HOME/.hiverc配置文件中加入`set hive.exec.mode.local.auto=true;`设置。特别是当我们常常处理的数据量不大的时候！

另：由于Hadoop运行的机器和Hive client运行的机器可能环境不一致，JVM版本的不同或者软件的libs不同会导致本地模式可能出现不可预期的错误。另外一点值得我们注意的是，本地模式运行在Hive Client上一个独立的子JVM上，如果你愿意，那么你可以通过`hive.mapred.local.mem`参数来设置该子JVM最大内存，默认情况下，这个参数的值为0，在这种情况下，Hive会让Hadoop决定子JVM的默认内存限制。

参考：  
[Hive GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-Hive,Map-ReduceandLocal-Mode)