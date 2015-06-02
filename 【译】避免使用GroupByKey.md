#【译】避免使用GroupByKey
译：LeoTse

原文：[Avoid GroupByKey](http://databricks.gitbooks.io/databricks-spark-knowledge-base/content/best_practices/prefer_reducebykey_over_groupbykey.html)

### 译文
让我们来看两个wordcount的例子，一个使用了`reduceByKey`，而另一个使用`groupByKey`:

`val words = Array("one", "two", "two", "three", "three", "three")`  
`val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))`  

`val wordCountsWithReduce = wordPairsRDD`  
&nbsp;&nbsp;&nbsp;&nbsp;`.reduceByKey(_ + _)`  
&nbsp;&nbsp;&nbsp;&nbsp;`.collect()`  

`val wordCountsWithGroup = wordPairsRDD`  
&nbsp;&nbsp;&nbsp;&nbsp;`.groupByKey()`  
&nbsp;&nbsp;&nbsp;&nbsp;`.map(t => (t._1, t._2.sum))`  
&nbsp;&nbsp;&nbsp;&nbsp;`.collect()`

上面两个函数所得到的结果都是正确的，但是当数据集很大时，使用了`reduceByKey`的例子表现更佳。这是因为在shuffle输出的数据前，Spark会Combine每一个partition上具有相同key的输出结果。

看下图我们就能理解`reduceByKey`的工作流程。我们注意到同一台机器上数据shuffle之前，相同key的数据（通过调用传入`reduceByKey`的lambda函数）Combine在一起的，然后再一次调用这个lambda函数去reduce来自各个partition的全部值，从而得到最终的结果。  

![ReduceByKey](http://databricks.gitbooks.io/databricks-spark-knowledge-base/content/images/reduce_by.png)


另一方面，当调用`groupByKey`的时候，所有的键值对都会进行shuffle，这将增加很多无谓的数据进行网络传输。

为了确定哪台机器将接受Shuffle后的键值对，Spark会针对该键值对数据的key调用一个分区函数。当某一台executor机器上的内存不足以保存过多的Shuffle后数据时，Spark就会溢写数据到磁盘上。然而，这种溢写磁盘会一次性将一个key的全部键值对数据写入磁盘，因此如果一个key拥有过多键值对数据——多到内存放不下时，将会抛出Out Of Memory异常。在之后发布的Spark中将会更加优雅地处理这种情况，使得这个job仍会继续运行，但是我们仍然需要避免（使用`groupByKey`）。**当Spark需要溢写磁盘的时候，它的性能将受到严重影响**。

![GroupByKey](http://databricks.gitbooks.io/databricks-spark-knowledge-base/content/images/group_by.png)


如果你有一个非常大的数据集，那么`reduceByKey`和`groupByKey`进行shuffle的数据量之间的差异将会更加夸张。

下面是一些你可以用来替代`groupByKey`的函数：  
1）当你在combine数据但是返回的数据类型因输入值的类型而异时，你可以使用`combineByKey`；  
2）如果key使用到结合函数和“零值”，你可以用`foldByKey`函数合并value；