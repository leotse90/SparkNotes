# RDD主要API简介
_By: LeoTse_

## Intro
RDD作为Spark的核心概念，它的相关操作是Spark学习者常见常用的。这里主要讨论RDD的一些常见的函数，并相应给出示例代码。想了解更多大家可以参考Spark源码RDD.scala。  
（下面的示例都是Python代码。）

## Transformations
### map(func, preservesPartitions=False)
这个函数的主要操作是针对RDD中的每一个元素，执行一次func函数的操作。我们看下面的示例：  

`rdd = sc.parallelize(["hello", "world", "hi", "world"])`  
`print rdd.map(lambda x:x+"-plus").collect()`  

**Output**：  
['hello-plus', 'world-plus', 'hi-plus', 'world-plus']

值得注意的是，新RDD和原来的RDD之间的元素是一一对应的关系。


### flatMap(func, preservesPartitions=False)
从函数名来看，直觉告诉我们和map()函数有关系，我们这次先看一个示例：  

`rdd = sc.parallelize(["hello", "world", "hi", "world"])`  
`print "flatMap result:"`  
`print rdd.flatMap(lambda x:(x, 1)).collect()`  
`print "map result:"`  
`print rdd.map(lambda x:(x, 1)).collect()`  

**Output**：  
`flatMap result:`  
`['hello', 1, 'world', 1, 'hi', 1, 'world', 1]`  
`map result:`  
`[('hello', 1), ('world', 1), ('hi', 1), ('world', 1)]`  

`map()`和`flatMap()`二者很类似，某种意义上讲，这两者都是线性遍历RDD的元素，然后针对每个元素执行func函数。两者的区别在于`map()`中的func函数只返回一个元素，而`flatMap()`中的func函数可以返回一个list。并且，`flatMap()`函数的输出是撑平的。值得注意的是，虽然`flatMap()`中的func函数返回list，但是`flatMap()`本身返回一个包含list中所有元素的、扁平化的RDD，而不是一个list。  
直白一点讲，那就是原RDD中的元素经过`map()`中func的转换后还是对应新RDD一个元素，而原RDD中的一个元素经过`flatMap()`中func函数转换后会对应新RDD中的一个list，但是`flatMap()`的输出会把所有的list撑平为一个list。


### mapPartitions(func, preservesPartitioning=False)
还是map一族的函数，这个函数的作用是对原RDD中的每一个partition调用func函数，并返回一个新的RDD：

`rdd = sc.parallelize(["leo", "tse", "handsome", "boy"], 2)`  
`def func(iter):`  
&nbsp;&nbsp;&nbsp;&nbsp;`yield " ".join(iter)`  

`print rdd.mapPartitions(func).collect()`  

**Output**：  
`['leo tse', 'handsome boy']`  

我们可以看到原RDD上有两个partition，我们分别对每一个partition执行func函数，它们的输出构成新RDD。也就是说`map()`的func处理的是一个元素，而`mapPartitions()`处理的是一个个partition。  
我们在RDD.scala中还看到`mapPartitions()`的其他类似物：  
**mapPartitionsWithIndex()**：把partition的index传递给func函数；


### filter(func)
这个函数我们用得比较多，当我们需要获取RDD中满足我们条件（即func函数）的元素并塞进一个新RDD然后返回。下面是一段简短的示例代码：    

`rdd = sc.parallelize(["hello", "world", "hi", "boy"])`  
`print rdd.filter(lambda x:"h" in x).collect()`

**Output**：  
`['hello', 'hi']`


### distinct()
### union(other)
### intersection(other)
上述三个函数的作用分别是去重，求多个RDD元素的并集，多个RDD元素的交集。简单示例代码如下：  

`rdd1 = sc.parallelize(["hello", "world", "world"])`  
`rdd2 = sc.parallelize(["hello", "leo", "tse"])`  

`print rdd1.union(rdd2).collect()`  
`print rdd1.intersection(rdd2).collect()`  
`print rdd1.distinct().collect()`  

**Output**：  
`['hello', 'world', 'world', 'hello', 'leo', 'tse']`  
`['hello']`  
`['world', 'hello']`  


### sortByKey(ascending=True, numPartitions=None, keyfunc=lambda x: x)
这个函数针对元素为键值对的RDD，根据键值对的key进行排序。ascending参数表示是升序还是降序，默认为升序，numPartitions为partition的个数，我们可以用keyfunc指定排序的规则：  

`test_dict_list = [("b", 3), ("C", 2), ("A", 5), ("d", 1)]`  
`dict_rdd = sc.parallelize(test_dict_list)`  

`print dict_rdd.sortByKey().collect()`  
`print dict_rdd.sortByKey(keyfunc=lambda x:x.lower()).collect()`  

**Output**：  
`[('A', 5), ('C', 2), ('b', 3), ('d', 1)]`  
`[('A', 5), ('b', 3), ('C', 2), ('d', 1)]`  

类似的函数还有**sortBy(keyfunc, ascending=True, numPartitions=None)**，keyfunc参数的位置居然和`sortByKey()`不一样，处女座表示不能忍！这是因为`sortBy()`函数需要用户必须指定排序的规则：  

`test_dict_list = [("b", 3), ("C", 2), ("A", 5), ("d", 1)]`  
`dict_rdd = sc.parallelize(test_dict_list)`  

`print dict_rdd.map(lambda x:x[1]).sortBy(lambda x:x).collect()`  

**Output**：  
`[1, 2, 3, 5]`  

当然，我们还是要注意`sortBy()`和`sortByKey()`两者的区别，后者针对的必须是元素为键值对的RDD。


### glom()
### cartesian(other)
`glom()`将不同分区的元素合并为一个Array，返回新的RDD。  
`cartesian(other)`则是计算两个RDD的[笛卡尔积](http://baike.baidu.com/view/348542.htm?fromtitle=%E7%AC%9B%E5%8D%A1%E5%B0%94%E7%A7%AF&fromid=1434391&type=syn)。

`rdd1 = sc.parallelize([1, 2], 2)`  
`rdd2 = sc.parallelize([3, 4])`  
`rdd3 = sc.parallelize([5, 6])`  

`print rdd1.glom().collect()`  
`print rdd2.cartesian(rdd3).collect()`  

**Output**：  
`[[1], [2]]`  
`[(3, 5), (3, 6), (4, 5), (4, 6)]`  


### groupBy(func, numPartitions=None)
这个类似于SQL里面的GroupBy，亦即对RDD中的元素根据给定的func函数进行分类。

`rdd = sc.parallelize([1, 2, 3, 4, 4, 5])`  
`rt = rdd.groupBy(lambda x:x%2).collect()`  

`print [(x, sorted(y)) for (x, y) in rt]`  

**Output**：  
`[(0, [2, 4, 4]), (1, [1, 3, 5])]`  


### zip(other)
类似于Python中的zip函数，合并两个RDD的元素为键值对形式，前提是两个RDD中元素个数一致。

`rdd1 = sc.parallelize([1, 2, 4])`  
`rdd2 = sc.parallelize(["a", "b", "c"])`  

`print rdd1.zip(rdd2).collect()`  

**Output**：  
`[(1, 'a'), (2, 'b'), (4, 'c')]`  


### join(other, numPartitions=None)
### leftOuterJoin(other, numPartitions=None)
### rightOuterJoin(other, numPartitions=None)
类似于SQL中的join操作。

`rdd1 = sc.parallelize([("a", 1), ("b", 2)])`  
`rdd2 = sc.parallelize([("a", 3), ("a", 4)])`  

`print rdd1.join(rdd2).collect()`  
`print rdd1.leftOuterJoin(rdd2).collect()`  
`print rdd1.rightOuterJoin(rdd2).collect()`  

**Output**：  
`[('a', (1, 4)), ('a', (1, 3))]`  
`[('a', (1, 3)), ('a', (1, 4)), ('b', (2, None))]`  
`[('a', (1, 3)), ('a', (1, 4))]`


### reduceByKey(func, numPartitions=None)
### groupByKey(numPartitions=None)
针对元素为键值对形式的RDD，合并相同key的键值对。

`rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 2), ("c", 2), ("a", 2)])`  

`print rdd.reduceByKey(lambda x,y:x+y).collect()`  

**Output**：  
`[('a', 5), ('c', 2), ('b', 1)]`

wordcount的即视感有木有！  
这两者还是有区别的，实际操作中我么应该尽量避免使用`groupByKey()`，具体可参考[【译】避免使用GroupByKey](https://github.com/leotse90/SparkNotes/blob/master/%E3%80%90%E8%AF%91%E3%80%91%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8GroupByKey.md)。

## Actions
### foreach(func)
### foreachPartition(func)
`foreach()`函数将func函数作用于RDD的每一个元素，而`foreachPartition()`将func函数作用于RDD的每个partition。

`def func(x):`  
&nbsp;&nbsp;&nbsp;&nbsp;`print x`  

`rdd = sc.parallelize([2, 4, 5)`  
`rdd.foreach(func)`  
`rdd.foreachPartition(func)`  


### reduce(func)
`reduce()`将RDD中元素两两传递给func函数，同时将新产生的值与RDD中下一个元素再传递给func函数直到最后只有一个值为止。 

`rdd = sc.parallelize([2, 4, 5, 7])`  
`print rdd.reduce(lambda x,y:x+y)`  

**Output**：  
`18`

上面的例子即把RDD中的每个元素相加。


### fold(zeroValue, op)
一种特殊的reduce函数，自带初始值zeroValue。op(t1, t2)函数只允许修改t1的值并返回结果作为新的t1，且不允许修改t2的值。


### collect()
这个函数我们的前面的很多示例代码中都用到了，它的作用是返回一个包含该RDD中所有元素的list。


### max()
### min()
### sum()
### count()
### mean()
### variance()
### stdev(), 
这一系列函数的作用很直观，分别是取RDD元素中最大值、最小值、求和、计数, 平均数，方差，标准偏差：

`rdd = sc.parallelize([2, 4, 5, 7])`  

`print rdd.max()`  
`print rdd.min()`  
`print rdd.sum()`  
`print rdd.count()`  
`print rdd.mean()`  
`print rdd.variance()`  
`print rdd.stdev()`  

**Output**：  
`7`  
`2`  
`18`  
`4`  
`4.5`  
`3.25`  
`1.80277563773`  


### zipWithIndex()
### zipWithUniqueId()
这两个函数和Transformation中的`zip()`函数作用类似，`zip()`函数是将两个RDD元素进行zip，而`zipWithIndex()`将RDD中元素与其本身的索引进行zip，`zipWithUniqueId()`则是为RDD中的每一个元素分配一个唯一的ID，而且如果该RDD中有n个partitions的时候，第k个partition的ID会呈现k, n+k, 2*n+k, ...规律递增。

`rdd = sc.parallelize(["a", "r", "e", "s"])`  

`print rdd.zipWithIndex().collect()`  
`print rdd.zipWithUniqueId().collect()`  

**Output**：  
`[('a', 0), ('r', 1), ('e', 2), ('s', 3)]`  
`[('a', 1), ('r', 3), ('e', 5), ('s', 7)]`  


### saveAsXXXX()
这一系列主要有`saveAsTextFile(path)`，`
saveAsSequenceFile(path, compressionCodecClass=None)`，`saveAsHadoopFile(path, outputFormatClass, keyClass=None, valueClass=None, keyConverter=None, valueConverter=None, conf=None, compressionCodecClass=None)`等等，这些函数的作用都是将RDD保存到外部存储。


## Conclusion
这里只是简单列举了一些RDD常见操作API，然后简单举例说明使用方法。具体的还是要结合实际灵活运用。不过理解了这些API，就能很轻松地对RDD进行处理。