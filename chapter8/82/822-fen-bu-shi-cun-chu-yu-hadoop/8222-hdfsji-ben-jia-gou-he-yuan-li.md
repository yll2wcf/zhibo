##8.2.2.2 HDFS基本架构和原理  

HDFS基本架构和原理  

HDFS架构图

图1  

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





