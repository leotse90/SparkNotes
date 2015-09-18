# Hive浮点数比较与计算
__author:leotse__

### Hive浮点数比较
Hive的较早前版本（0.9.0）在比较浮点数时会出现问题，如下：  
`hive> select * from float_compare where weight>120.9;`  
`OK`  
`xiefeng 128.2`  
`yolovon 121.8`  
`leotse 120.9`  
我们可以看到，leotse乱入了，他的weight等于120.9，这一点与我们的预期不符合。设想一下，如果这些数据不是weight而是money，那么结果就会变成后果。这是Hive的一个BUG，我们在Hive的issues上可以看到[Float comparison doesn't work](https://issues.apache.org/jira/browse/HIVE-2586)。这个问题已经解决了，但是我们仍然可以分析一下其他的原因。

在float_weight表中，weight以float类型保存。当我们在HiveQL语句输入120.9时，Hive会将其保存为double类型，这样我们看看120.9分别在这两种类型的表示：  
`float 120.9000001`  
`double 120.900000000001`  
我们知道，在比较float和double类型数值时，float会强制转型为double。这样当float_compare表中数值120.9在和HQL中120.9比较时，前者会转型为120.900000100000。我们可以看到120.900000100000>120.900000000001。这也就是为什么leotse会出现在查询结果的原因。

虽然这个问题已经解决，但是仍然给我们一些启示，当我们在进行浮点数比较的时候，需要警惕float和double的自动转型，特别需要避免从窄类型数值向宽类型数值的转型。

### Hive浮点数计算
相比上面的问题，浮点数的计算也是一个BUG，而且现在还没有解决，[float and double calculation is inaccurate in Hive](https://issues.apache.org/jira/browse/HIVE-3715)。

为了方便对比，我们先展示出表float_compare中的全部数据：  
`hive> select * from float_compare;`   
`OK`   
`dooley 110.0`   
`xiefeng 128.2`   
`leotse	120.9`  
`februus 119.0`  
`yolovon 121.8`  

接下来我们将表中的weight的字段除以10，得到如下结果：  
`hive> select weight/10 from float_compare;`  
`OK`  
`11.0`  
`12.819999694824219`  
`12.09000015258789`  
`11.9`  
`12.180000305175781`  

我们可以看到，除了110.0和120.0这种小数点后为0的，其他浮点数计算的结果都是不符合我们预期的。实际上，这个问题在Hive和Java中都存在（Hive用Java实现），而且所有使用IEEE标准进行浮点数编码的系统都存在这个问题。目前这个没有被解决，在这里摆出来是希望以后在遇到类似的浮点数计算的时候，我们能够长一个心眼，以免出现了谬误还不知道为什么。