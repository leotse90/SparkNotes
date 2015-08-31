# Hive笔记（四）
__整理：leotse__

### 分区表
1.**数据分区**：通常使用分区来水平分散压力，将数据从物理上转移到和使用最频繁的用户更近的地方，以及实现其他目的。

2.Hive中有分区表的概念。分区表有重要的性能优势，还可以降数据以一种符合逻辑的方式进行组织。

3.分区管理表：  
`CREATE TABLE student(`  
`name STRING,`  
`schoold_id STRING)`  
`PARTITIONED BY (grade STRING, class STRING);`  
分区表改变了Hive对数据存储的组织方式。Hive将会创建好可以反映分区结构的子目录。grade和class已经包含在文件目录名称中了，所以也就没有必要将这些值存放在它们目录下的文件中了。

**Note**：怎么定义这些分区字段，主要是看我们查询的需求，而且这些字段变成分区字段的另一个好处就是节省空间。

4.对数据进行分区，也许最重要的原因就是为了更快地查询。  
当我们在WHERE子句中增加谓词来按照分区值进行过滤时，这些谓词称为**分区过滤器**。比如：  
`SELECT * FROM student WHERE grade='一年级' AND class='101班';`  

**Note**：如果用户查询全部的数据（不按分区查询），假设我们所有的数据量非常大，这是就会触发一个巨大的MapReduce任务，一个高度建议的安全措施就是将Hive设置为**“strict”模式**，这样如果对分区表进行查询而WHERE子句**没有加分区过滤的话，将会禁止提交这个任务**：  
`set hive.mapred.mode=strict/nonstrict;`

查看所有分区可以使用下面命令：  
`SHOW PARTITIONS student;`

5.**外部分区表**：外部表同样可以使用分区，这种结合可以为用户提供一个可以和其他工具共享数据的方式，同时也可以优化查询性能。

**Note**：一个非常合适的场景就是日志文件分析：  
`CREATE EXTERNAL TABLE IF NOT EXISTS log_message(`  
`hms INT,`  
`severity STRING,`  
`server STRING,`  
`process_id INT,`  
`message STRING)`  
`PARTITIONED BY (year INT, month INT, day INT);`

6.Hive不关心一个分区对应的分区目录是否存在或者分区目录下是否有文件。如果分区目录不存在或者分区目录下没有文件，则对于这个过滤分区的查询将没有返回结果。

7.外部分区表和普通外部表一样，Hive并不控制数据，即使表被删除，数据也不会被删除。

### 自定义表的存储格式
1.Hive默认的存储格式是文本文件，这也可以使用子句STORED AS TEXTFILE来指定。

2.我们来看如下建表语句：  
`CREATE TABLE student(`  
`name STRING,`  
`schoold_id STRING)`  
`ROW FORMAT DELIMITED`  
`FIELDS TERMINATED BY '\001'`  
`COLLECTION ITEMS TERMINATED BY '\002'`  
`MAP KEYS TERMINATED BY '\003'`  
`LINES TERMINATED BY '\n'`   
`STORED AS TEXTFILE;`  
  
TEXTFILE：意味着所有字段都是使用字母、数字、字符编码，包括那些国际字符集，使用TEXTFILE就会隐式规定每一行都是一个单独的记录；我们还可以将TEXTFILE替换为其他Hive所支持的内置文件格式，包括SEQUENCEFILE和RCFFILE。    
用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe

记录的编码：通过一个inputformat对象来控制；  
记录的解析：由序列化器／反序列化器来控制；

### 删除表&修改表
1.Hive使用DROP TABLE来删除表。

**Note**：如果用户开启了Hadoop回收站功能（默认关闭），那么数据将会转移到用户在分布式文件系统中的用户根目录的.Trash目录下。如果不小心删除了一张存储着重要数据的管理表，那么可以先重建表，然后重建所需的分区，再从.Trash目录中将误删的文件移动到正确的文件目录下来重新存储数据。

2.ALTER TABLE仅仅会修改表元数据，表数据本身不会有任何修改。需要用户自己确认所有的修改都和真实的数据是一致的。

3.**表重命名**:`ALTER TABLE table_1 RENAME TO table_2;`  
**增加分区**:`ALTER TABLE table_1 ADD PARTITION ...;`  
**删除分区**:`ALTER TABLE table_1 DROP IF EXISTS PARTITION(year=2011, month=4, day=12);`  
**修改列信息*:`ALTER TABLE table_1 CHANGE COLUMN col1 INT;`   
**增加列**:`ALTER TABLE table_1 ADD COLUMN (col1 INT, col2 STRING);`  

