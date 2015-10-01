# HDFS文件系统命名空间
_整理:leotse_

### HDFS Namespace
在HDFS中，我们知道NameNode负责管理文件系统的命名空间，那么NameNode到底怎么管理HDFS的命名空间，又有哪些内容需要管理呢？我们接下来将讨论到这两个问题。

作为HDFS的Master，NameNode掌握着整个HDFS的文件目录树及其目录与文件，这些信息会以文件的形式永久地存储在本地磁盘。我们可以在$HADOOP_HOME/tmp/dfs/name/current下找到这些文件：fsimage以及edits。

**fsimage**：保存了最新的元数据检查点；  
**edits**：保存了HDFS中自最新的元数据检查点后的命名空间变化记录；

为了防止edits中保存的最新变更记录过大，HDFS会定期合并fsimage和edits文件形成新的fsimage文件，然后重新记录edits文件。由于NameNode存在单点问题（Hadoop2.0以前版本），因此为了减少NameNode的压力，HDFS把fsimage和edits的合并的工作放到SecondaryNameNode上，然后将合并后的文件返回给NameNode。但是，这也会造成一个新的问题，当NameNode宕机，那么NameNode中edits的记录就会丢失。也就是说，NameNode中的命名空间信息有可能发生丢失。

### FsImage
Fsimage是一个二进制文件，它记录了HDFS中所有文件和目录的元数据信息。关于fsimage的内部结构我们可以参看下图：  
![fsimage](https://github.com/leotse90/SparkNotes/blob/master/images/fsimage.jpg)  
乍看之下，这张图有点晦涩难懂，这张图是fsimage的内部结构，第一行是文件系统元数据，第二行是目录的元数据信息，第三行是文件的元数据信息，下面将逐个解析图中的字段：  
**imgVersion**：当前fsiamge文件的版本号；  
**namespaceID**：当前命名空间的ID，在NameNode的生命周期内保持不变，DataNode注册时，返回该ID作为其registrationID，每次和NameNode通信时都要检查，不认识的namespaceID拒绝连接；  
**numFiles**：文件系统中的文件数；  
**genStamp**：生成该fsimage文件的时间戳；  
**path**：文件或者目录路径；  
**replicas**：文件的副本数，目录的replicas为0；  
**mtime**：修改时间；  
**atime**：访问时间；  
**blocksiz**：文件的块size，目录的size为0；  
**numBlock**：文件包含的数据块数量，目录的为－1；  
**nsQuota**：目录的命名空间大小配额，默认为－1；  
**dsQuota**：目录的磁盘大小配额，默认为－1；  
**username**：文件或者目录所属的用户名；  
**group**：用户所属的组名；  
**perm**：即permission，访问权限；  
**blockid**：文件的文件块id；  
**numBytes**：该文件块的bytes数，即文件块的大小；  
**genStamp**：该文件块的时间戳。

那么怎么在fsimage中保存根目录呢？path的length为0，即表示这个目录为根目录。

NameNode将这些信息读入内存之后，构造一个文件目录结构树，将表示文件或目录的节点填入到结构中。

### BlocksMap
我们知道，NameNode将文件命名空间的文件树结构等信息固化在本地文件中，同时还将文件块与DataNode的映射关系存储在内存中。NameNode是通过DataNode的blockreport获取文件块与DataNode的映射关系的。

BlocksMap中保存了文件块block与DataNodes的映射信息以及DataNode与文件块blocks的信息，这里用到三元组进行表示，每个文件块block有几个副本，就有几个三元组：  
`(DataNodeID, PreBlock, NextBlock)`  
第一个元素DataNodeID表示当前文件块block存储在哪个DataNode上；第二个元素PreBlock指向前一个文件块block；第三个元素NextBlock指向下一个文件块block；

借助这个三元组可以找到一个文件块block所属的所有DataNode，也可以通过三元组的后两个元素信息找到一个DataNode上所有的文件块blocks。 

### Conclusion
通过fsimage与blocksmap两种数据结构，NameNode就能建立起完整的命名空间信息以及文件块映射信息。在NameNode加载fsimage之后，BlocksMap中只有每个block到其所属的DataNode列表的对应关系信息还没建立，这个需要通过DataNode的blockReport来收集构建，当所有的DataNode上报给NameNode的blockReport处理完毕后，BlocksMap整个结构也就构建完成。

NameNode在format时生成fsimage/edits/Shared Edits文件，而且它们都需要格式化并且通过clusterId保持一致。