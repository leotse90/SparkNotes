# Hive之JOIN操作
_整理：leotse_

### Introduction
在MySQL等关系型数据库中，我们经常会用到联表查询，用以查询多张表的数据。  
使用联表查询的场景一般是：我们需要对两张或多张表中的数据进行关联，从而得到我们所需要的数据集。  

在Hive中，也存在这样的场景，特别是进行数据分析的时候，但是，Hive虽然支持Hive JOIN语句，但是**只限于等值连接**，也就是说，只支持a.col=b.col而不支持a.col >= b.col这类的条件，主要原因是通过MapReduce很难实现这种类型的联接。

接下来，我们将分析和介绍Hive中主要的JOIN操作。

我们假定我们有两张表：human和assets。它们分别记录了人口以及资产信息，它们的字段如下：  
`hive> describe human;`  
`OK`  
`name                	string`             	                    
`age                 	int`         	                    
`job                 	string`                	                    
`addr                	string`  

`hive> describe assets;`  
`OK`  
`name                	string`                	                    
`bank                	string`                	                    
`fund                	int`  

在下面的例子中，将会用到这两张表。

### INNER JOIN
INNER JOIN，又称内连接。是较为常见的一种JOIN操作：  
**Input**: 表A、表B  
**Condition**: 表A和表B中符合我们期望的，A.col = B.col   
**Output**: 表A和表B中满足Condition的所有数据记录   
**Example**:   
`SELECT a.col1, a.col2, b.col2, b.col3`  
`FROM A a JOIN B b`  
`ON a.col1 = b.col1`  
`WHERE a.col3 = 'CONDITION';`  

我们来看一个具体的例子，我们想知道human表中每一个人在不同银行的存款记录，我们可以用以下SQL语句进行查询：  
`SELECT h.name, h.age, h.job, a.bank, a.fund FROM human h JOIN assets a ON h.name=a.name;`  
ON关键字指定了两种表联接的条件，
我们对输出结果进行了截取，只选取最终的数据查询记录，如下：  
`xiefeng	24	big data	CMB	15000000`  
`leotse	25	programmer	CMB	200000000`  
`leotse	25	programmer	CAB	10000000`   
`yolovon	26	CEO	HB	5000000`  
`yolovon	26	CEO	CCB	3200000`  

另外，我们需要注意的是，在Hive中，ON子句中尚不支持谓词OR。

如果我们对多表进行INNER JOIN操作，如:  
`SELECT a.col1, a.col2, b.col4, c.col5 FROM a`  
`JOIN b ON a.col1 = b.col2`  
`JOIN c ON a.col2 = c.col3`  
`WHERE CLAUSE;`  
大多数情况下，Hive会对每次JOIN操作启动一个MR Job，（在Hive中，SQL的执行是从左至右的）这上面这个SQL语句的执行过程中，首先启动一个MR Job对表a和表b进行一次JOIN操作，然后再启动另一个新的MR Job对表a和表c进行一次JOIN操作。  
上面说到是大多数情况下，那种小部分的情况是指只要我们使用的ON后的条件（即连接键）一致的话，那就只会产生一个MR Job。

Hive还假定最后一张JOIN的表最大，从而先将前面的表先缓存起来，直到最后一张表才进行计算，因此为了防止内存消耗过大，我们应该尽量保证JOIN操作的表从左到右表大小递增。

