# Hadoop常见问题
__整理：LeoTse__

### MySQL字符集（一）
1.问题：  
`com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Specified key was too long; max key length is 767 bytes.`   

2.原因：  
保存Hive元信息的数据库字符集问题，hive保存元信息的数据库字符集需要是Latin1。  

3.解决方案：  
改变数据库的字符集。  
`alter database metastore_db character set latin1;`

### MySQL字符集（二）
1.问题：  
`FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:For direct MetaStore DB connections, we don't support retries at the client level.)`  

2.原因：  
查看MySQL中hive元数据数据库的字符集，如果不是Latin1，修改即可。

3.解决方案：  
修改Hive元信息数据库的字符集：  
`alter database metastore_db character set latin1;`

### MySQL字符集（三）
1.问题：  
drop table 或者 drop database的时候一直卡住。（注意：删除一个DB时需要确保里面没有数据，如果需强行删除，加上cascade）。  

2.原因：  
重要的错误来三遍，MySQL字符集的问题。   

3.解决方案：  
按照上面的修改字符集为latin1即可。  
建议在第一次启动hive前，手动建库metastore_db，并指定字符集为latin1。这样可以避免后续出现的一系列问题。


### MySQL复制模式
1.问题：  
`Caused by: java.sql.SQLException: Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging. InnoDB is limited to row-logging when transaction isolation level is READ COMMITTED or READ UNCOMMITTED.`    

2.原因：  
MySQL复制模式主要有STATEMENT、ROW以及MIXED三种模式。MySQL默认使用的是STATEMENT，这种模式在处理事务、UDF等情况下会出现问题。  
这里的错误可以看到，由于MySQL 用的引擎为InnoDB, STATEMENT只支持事务的隔离级别是REPEATABLE READ或者SERIALIZABLE，其他的都不行。  
了解更多MySQL Replication相关的知识，可以参考：  
[Replication Formats](http://dev.mysql.com/doc/refman/5.7/en/replication-formats.html)   
[Mixed Binary Logging Format](http://dev.mysql.com/doc/refman/5.7/en/binary-log-mixed.html)   
[Advantages and Disadvantages of Statement-Based and Row-Based Replication](http://dev.mysql.com/doc/refman/5.7/en/replication-sbr-rbr.html)    

3.解决方案：  
将binlog_format修改为MIXED模式，可以在my.cnf文件中修改`binlog_format=MIXED`（如果没有，增加即可），然后重启mysql数据库。如果仍然报该错误，删除存储元信息的相关表（如果存储已经有数据，不建议这样做）。

### NodeManager未启动
1.问题：  
Hive 启动MR job后，长期未响应，查看JOB的状态发现：  
`FinalStatus:	UNDEFINED`  

2.原因：  
执行jps发现，没有NodeManager，因此无法提交JOB，也就没有办法进一步执行。

3.解决方案：  
启动NodeManager即可：  
`sbin/yarn-daemon.sh start nodemanager`  

### SafeMode
1.问题：  
启动hive失败，异常中出现如下提示：  
`Name node is in safe mode.`  

2.原因：  
在分布式文件系统启动的时候，开始的时候会有安全模式，当分布式文件系统处于安全模式的情况下，文件系统中的内容不允许修改也不允许删除，直到安全模式结束。安全模式主要是为了系统启动的时候检查各个DataNode上数据块的有效性，同时根据策略必要的复制或者删除部分数据块。运行期通过命令也可以进入安全模式。在实践过程中，系统启动的时候去修改和删除文件也会有安全模式不允许修改的出错提示，只需要等待一会儿即可。

3.解决方案：  
我们选择等待检查完毕，也可以使用以下命令直接解除安全模式：  
`hadoop dfsadmin -safemode leave`  

### Connection Refused
1.问题：  
`Application application_1441163257106_0002 failed 2 times due to Error launching appattempt_1441163257106_0002_000002. Got exception: java.net.ConnectException: Call From Master/172.16.10.42 to localhost:46940 failed on connection exception: java.net.ConnectException: 拒绝连接`

2.原因：  
查看这个异常，我们可以知道是因为连接不到Master的端口，具体可参看[这里](http://wiki.apache.org/hadoop/ConnectionRefused)。

3.解决方案：  
hosts配置问题，localhost:46940不能访问。注释掉/etc/hosts中的`IP localhost`。

### ClusterID不一致
1.问题：  
`FATAL org.apache.hadoop.hdfs.server.datanode.DataNode: Initialization failed for Block pool <registering> (Datanode Uuid unassigned) service to Master/10.14.13.210:9000. Exiting.`  
`java.io.IOException: Incompatible clusterIDs in /home/xiefeng/packages/hadoop-2.6.0/tmp/dfs/data: namenode clusterID = CID-3cab9049-42dd-4a70-8248-32c3cbeefaed; datanode clusterID = CID-f13dbf4f-7625-4a19-91ac-cac5fae58443`  
	`at org.apache.hadoop.hdfs.server.datanode.DataStorage.doTransition(DataStorage.java:648)`  
	`at org.apache.hadoop.hdfs.server.datanode.DataStorage.addStorageLocations(DataStorage.java:320)`  
	`at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:403)`  
	`at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:422)`  
	`at org.apache.hadoop.hdfs.server.datanode.DataNode.initStorage(DataNode.java:1311)`  
	`at org.apache.hadoop.hdfs.server.datanode.DataNode.initBlockPool(DataNode.java:1276)`  
	`at org.apache.hadoop.hdfs.server.datanode.BPOfferService.verifyAndSetNamespaceInfo(BPOfferService.java:314)`  
	`at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:220)`  
	`at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:828)`  
	`at java.lang.Thread.run(Thread.java:745)`  

2.原因：  
datanode的clusterID 和 namenode的clusterID 不匹配。

3.解决方案：  
首先暂停服务；然后将namenode的VERSION的clusterID复制到datanode的VERSION中。  
namenode的VERSION在/home/xiefeng/packages/hadoop-2.6.0/tmp/dfs/name目录下；  
datanode的VERSION在/home/xiefeng/packages/hadoop-2.6.0/tmp/dfs/data目录下。

