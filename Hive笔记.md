# Hive笔记
__整理:leotse__

### Hive的基本原理
1.Hive最适合数据仓库应用程序，使用该程序进行相关的静态数据分析，不需要快速给出结果，而且数据本身不会频繁变化。

**Note**：这是Hive的应用场景，由于Hive本身只是HDFS的结构化，即在HDFS的data上加上schema，这决定了它的应用场景以及其局限性。

2.Hive的所有查询和命令都会进入到Driver（驱动模块），通过该模块对输入进行解析编译，对需求的计算进行优化，然后按照指定的步骤执行。当需要启动MR时，Hive本身是不会生成Java MR算法程序的。相反，Hive通过一个表示“job执行计划”的XML文件驱动内置的、原生的Mapper和Reducer模块。

**Note**：我们都知道Hive SQL的执行是通过将SQL方言转换为MR程序来完成，上面是Hive SQL转换的基本思路，具体的实现我们可以酌情去选择了解。

Hive通过和JobTracker通信来初始化MR任务（Job），而不必部署在JobTracker所在的节点上执行。

3.MetaStore（元数据存储）是一个独立的关系型数据库，Hive会在其中保存表模式和其他系统元数据。  
所有的Hive客户端都需要一个metastoreservice（元数据服务），Hive使用这个服务来存储表模式信息以及其他元数据信息。通常情况下会使用一个关系型数据库中的表来存储这些信息。默认情况下会使用内置的DerbySQL服务器，其可以提供有限的、单进程的存储服务。例如，当使用DerbySQL时，用户不可以执行2个并发的Hive CLI实例，对于集群来说，需要使用MySQL或者类似的关系型数据库。

**Note**：之所以需要将元数据保存在其他关系型数据库，是因为Hive的元信息需要不断的变更、修改，因此不将它们保存在HDFS中。

### Hive的使用
1.Hive中一次使用的命令：hive －e ｛sql script｝。执行结束后hive CLI立即退出。需要注意的是，Hive会将输出写到标准输出中。

2.Hive从文件中执行Hive查询：hive －f ｛file path｝。按照惯例，一般把这些Hive查询文件保存为具有.q或.hql后缀名的文件。

3.Hive查看操作命令历史，Hive将最近的10,000条命令记录到文件$HOME/.hivehistory中。