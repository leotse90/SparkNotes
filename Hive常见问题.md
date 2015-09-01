# Hive常见问题
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