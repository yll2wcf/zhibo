##4.3.2 分布式存储与HDFS  
在我们的系统环境中，需要保存用户大量的录制文件。传统的数据库显然不能满足我们的需求，我们需要一种跨机器、统一管理的分布式文件系统。HDFS是Hadoop Distributed File System的简称，源自于Google的GFS论文，是GFS的开源克隆版，至今已10年左右历史。我们的直播系统采用HDFS来存储海量文件。本节我们简单介绍一下HDFS以及与它相关的Hadoop组件。
###1 HDFS概述  
HDFS是易于扩展的分布式文件系统 运行在大量普通廉价机器上，所以成本较低。比如目前有10台机器，能够提供20T存储能力。随着业务量不断增长，只需要简单增加机器，配置好，就可以了，而不需要购买昂贵的存储设备。HDFS设计之初就非常明确其应用场景，适用于什么类型的应用，不适用什么应用，有一个相对明确的指导原则。下面列举一些HDFS的特点。  
#### 高容错性
HDFS将一份文件复制成多个副本存储来提供容错机制。当某几台设备出现故障，文件不会丢失。同时，会在其他未故障设备上自动保存缺失的副本。  
#### 适合大数据批处理，移动计算不移动数据
HDFS适合存储非常大的文件，每个文件的元数据信息是保存在NameNode的内存中，整个文件系统的文件数量会受限于NameNode的内存大小。大量的文件数量会耗尽NameNode的内存而使整个系统不可用。实际中更多是将小文件合并成大文件来存储。HDFS适用于数据的批处理，而不太适合低延迟的数据访问，它不像数据库有索引，HDFS是在高吞吐和低延迟做了一个折中。  当有计算任务提交到HDFS时，计算数据在哪个节点上，就在那个节点上执行任务，HDFS不会将数据拷贝到任务所在节点执行，以减少数据在网络之间的传输。  
#### 流式数据访问  
HDFS基于这样的一个假设：最有效的数据处理模式是一次写入、多次读取。HDFS不适合并发写入，一个文件同一时刻只能有一个写入者。一般写入HDFS的文件就不再更改了，更改的话也只能在文件末尾追加，HDFS不提供文件随机修改。实际应用中我们会生成大量的日志数据，这些数据一般都不会修改，可以保存在HDFS中，每个文件被切成多块，由多副本保存，保证数据的一致性。
#### 构建成本低，安全可靠
HDFS不需要特别贵的机器，大型机售价非常高，动辄上百万甚至上千万。HDFS可运行于普通商用机器，对小公司来说性价比是非常高的。实际应用中HDFS已经得到了工业界的广发验证，根据Hadoop官网显示，Yahoo！的Hadoop集群约有10万颗CPU，运行在4万个机器节点上。Hadoop能流行其最主要的原因是花小钱可以办大事。  
###2 HDFS基本架构和原理  
先来看一下HDFS的整体架构，如图4-19。  
![](/assets/HDFS架构.png)  
图4-19

典型的master-slave架构
master-NameNode 命名空间，类似linux系统的文件目录树。默认3个副本。 NameNode高可用  通过ZK选举active  
slave-DataNode 真正的数据块存储。客户端读取文件，先去namenode读取文件元信息，拿到文件blcok所在datanode的地址。然后去datanode读取文件。  

核心概念  

1、Active namenode
主master（只有一个）
管理HDFS文件系统的命名空间（永久保存在磁盘中，为了快速访问，加载到内存中 fsimage）
维护文件元数据信息
管理副本策略（默认3个）
处理客户端读写请求

2、Standby namenode
active namenode的热备节点
周期性同步edits编辑日志，定期合并fsimage与edits到本地磁盘
active namenode故障时快速切换为新的active  

3、Namenode元数据文件  
edits：编辑日志，客户端对目录和文件的写操作首先被记到edits日志中，如：创建文件、删除文件等
fsimage：文件系统元数据检查点镜像文件，保存了文件系统中所有的目录和文件信息，如：一个目录下有哪些子目录、子文件，文件名，文件副本数，文件由哪些块组成等。
namenode内存中保存一份最新的镜像信息 镜像内容=fsimage+edits
namenode定期将内存中的新增的edits与fsimage合并保存到磁盘

4、Datanode  
Slave工作节点，可以启动多个。轻松的增加datanode来扩容。  
存储数据和数据校验和，来查看数据块是否损坏。如果损坏会向其他datanode读取。  
执行客户端的读写请求
通过心跳机制定期向namenode汇报运行状态和所有快列表信息
在集群启动时datanode向namenode提供存储的block块列表信息  

5、Block数据块
文件写入到HDFS会被切分成若干个Block块
数据块大小固定，默认大小128M，可自定义修改
HDFS最小存储单元
若一个块的大小小于设置的数据块大小，则不会占用整个块的空间
默认情况下每个Block有三个副本

6、Client
文件切分
与Namenode交互获取文件元数据信息
与Datanode交互，读取或写入数据
通过API或者管理命令来管理HDFS

HDFS为什么不适合存储小文件
元数据信息存储在namenode内存中，内存大小有限，也就是namenode存储block数目有限
一个block元信息消耗大约150byte内存
存储1亿个block，大约需要20G内存
如果一个文件大小为10K，则1亿个文件大小仅有1TB，却消耗namenode20GB内存

HDFS高可用原理

图

1）元数据同步
当有写请求发送到active namenode（NN1）时，NN1首先将写请求写入本地磁盘，然后同步阻塞写入JN（JournalNode）的edit。JournalNode是一组独立部署的服务器，专门用来存储NN的edit日志。与zookeeper类似，JN一般部署奇数台，当有过半数机器完成写操作即返回成功。NN1完成本地和JN的写操作后，会更新内存edit，同时与内存中的fsimage镜像合并，定期将合并后的fsimage写入磁盘（持久化）。Standby Namenode（NN2）定期同步JN的edit，之后与NN1一样在内存中合并fsimage和edit并持久化。在Datanode完成真正的数据块写入操作后，会定期向所有的namenode汇报数据块位置信息，这样就完成了元数据的同步。
2）主备切换
在每个namenode启动的时候，都会启动HealthMonitor和ActiveStandbyElector两个进程。ActiveStandbyElector进程就是负责主备切换的，它们会去zookeeper创建相同的临时节点，根据zookeeper的原理，只能有一个ActiveStandbyElector创建成功。创建成功的ActiveStandbyElector所属的namenode即为active namenode。创建失败的ActiveStandbyElector则会监听该临时节点。


###3 YARN基本架构和原理








