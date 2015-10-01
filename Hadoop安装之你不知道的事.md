#Hadoop安装之你不知道的事
_整理:leotse_

## 概述
这不是一篇教你怎么搭建Hadoop集群的文章，如果你需要安装Hadoop，可以直接跳转到[这里](https://github.com/leotse90/blogs/blob/master/Hadoop集群部署(RedHat).md)。

那么这篇文章究竟在讲些什么呢？我们讨论一些在搭建Hadoop集群时的细节。比如：NameNode的format干了些什么？又比如为什么我们需要在搭建集群前设置SSH免密码登录？诸如此类的都在我们这里的讨论范围之内。

我们先简单总结一下Hadoop集群的安装过程：  
1.准备工作：SSH免密码登录设置以及HostName配置；  
2.NameNode安装与配置：解压Hadoop安装包到NameNode服务器上，并修改配置文件；  
3.DataNode安装与配置：将NameNode的Hadoop文件分发到各DataNode服务器；  
4.集群启动：格式化NameNode，然后先后启动Hadoop集群的dfs以及yarn；

## Hadoop集群搭建的那些事
### 为什么需要进行SSH免密码登录的设置？
我们都知道SSH免密码登录可以让服务器之间在SSH登录时畅通无阻。那么，我们在Hadoop集群中，哪些过程需要用到SSH登录呢？  
首先，是DataNode与DataNode之间的文件传输；  
其次，在Hadoop的启动过程中，需要登录到其他机器的sbin目录下执行一些文件。

这样，为了方便Hadoop运作，我们建议事先在集群的所有机器进行SSH免密码登录设置。  
你可能注意到，我们在这里只是建议设置，并没有强制进行SSH免密码登录，因为我们只是为了方便Hadoop集群的运作如此设置，也就是说就算不这样设置我们也能搭建和运转Hadoop集群。

### 为什么需要配置Hostname？
这个的目的就更加简单了，只要配置了Hostname，我们就能较为灵活地进行配置，就算IP发生了变更，也不用改变所有服务器上所有的配置文件。

### 要修改哪些配置文件？
刚开始搭建Hadoop集群时，总是会被其复杂的配置文件弄得头昏脑胀；那么多的文件，有哪些是需要我们配置的？每个文件又是干嘛的？我们要配置哪些字段？还有一些相关的问题，如果我们想要解答这些问题，我们就需要系统了解一下Hadoop的构造，Hadoop主要有HDFS、Yarn以及MR，分别对应Hadoop集群的三大功能：数据存储、集群管理以及数据计算。这些功能模块分别对应hdfs-site.xml、yarn-site.xml和mapred-site.xml三个配置文件，这样我们就逐渐清晰了我们需要调整和配置的文件。

另外，我们需要告诉Hadoop中的Master角色服务器了解集群有哪些Slave机器，因此我们还需要配置一下Slaves文件。这个文件我们常常结合Hostname中的配置进行设置。

### NameNode在format时都干了些什么？
我在[另一篇文章](https://github.com/leotse90/SparkNotes/blob/master/HDFS文件系统命名空间.md)的最后提到了NameNode会在format的时候会创建fsimage、edits等文件，这些文件作为Hadoop集群HDFS的文件目录树及其目录和文件。

## 结语
一开始接触Hadoop的时候，搭建集群以及一些细枝末节的问题常常会成为我们掌握和驾驭Hadoop的障碍，我们除了简单粗暴直接上手搭建之外，也需要了解Hadoop集群内部的一些设计理念以及实现思路，这是我们学习和掌握Hadoop的必要储备。这里讲到了一些有关Hadoop集群搭建配置的相关信息，虽然不够全面，但是也希望能帮助大家通过Hadoop的搭建了解更真实的Hadoop集群。