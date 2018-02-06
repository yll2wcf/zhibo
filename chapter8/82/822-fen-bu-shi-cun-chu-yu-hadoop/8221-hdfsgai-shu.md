##8.2.2.1 HDFS概述  
HDFS概述
HDFS源自于Google的GFS论文，发表于2003年10月。Hadoop至今已经10年左右历史，开源的。HDF是GFS的克隆版
Hadoop Distributed File System 易于扩展的分布式文件系统 运行在大量普通廉价机器上，成本低。提供容错机制  
比如目前有10台机器，能够提供20T存储能力。随着业务量不断增长，简单增加机器，配置好，就可以了。  

多副本提供容错机制。某几台机器出现故障，文件不会丢失。

优点：
高容错性：数据自动保存过个副本、副本丢失后，自动恢复。
适合大数据批处理：  
移动计算不移动数据：计算任务提交到hadoop时，看数据在哪个节点上，就在那个节点上执行任务，不会将数据拷贝到任务所在节点执行，减少数据在网络之间的传输

数据位置暴露给计算框架：提供简单的API接口，通过接口查询到数据所在位置。  
GBTB甚至PB级别数据  
百万规模以上的文件数量  
10K节点规模 上万台，得到了工业界的验证  

流式数据访问：一次写入，一般写入就不再更改了（只能追加，不能更新）多次读取。每个文件被切成多块，由多副本保存。保证数据一致性  


构建成本低、安全可靠：
构建在廉价机器上、大型机售价非常高。上百万甚至上千万。对小公司来说性价比是非常高的。通过多副本提高可靠性、提供了容错和恢复机制
Hadoop能流行主要是因为花小钱办大事

缺点：
不适合低延迟数据访问：不像数据库有索引，在高吞吐和低延迟做了一个折中。
不适合大量小文件存储：占用NameNode大量内存空间（元数据），实际中更多是将小文件合并成大文件来存储。磁盘寻d道时间超过读取时间

不适合并发写入：一个文件只能有一个写入者
不提供文件随机修改：只支持追加  
生成大量的日志数据，一般都不会修改。

