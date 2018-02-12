##4.3.3 分布式消息队列与Kafka  
###1 什么是消息队列  
我们通过一个故事来引入消息队列的含义。  
比如，我们要做一个用户注册接口，同时在用户注册成功后给用户发一封成功邮件。很简单，是不是？提供一个注册接口，保存用户信息，同时发起邮件调用，待邮件发送成功后，返回用户操作成功。这是一种单线程同步阻塞的方式。用户使用以后反馈注册操作响应太慢。聪明的你一定想到我们可以用多线程啊，主线程直接返回注册结果，另起线程进行邮件发送。OK，这样就没问题了吗？有用户反应，邮件收不到。能否在发送邮件失败时，进行补发。这时候有人跟你说，邮件服务这块，别的团队都已经做好了，你不用再自己搞了，直接用他们的服务。你赶紧去和邮件团队沟通，谁知他们的服务根本就不对外开放，没有提供调用接口。明知道有这么一个服务，可是偏偏又调用不了。怎么办？邮件团队的人又跟你说，对外接口我们虽然不提供，但是我可以给你提供了一个类似邮局信箱的东西，你往这信箱里写上你要发送的消息，以及要发送的地址。之后你就不用再操心了，我们自然能从信箱中取得地址和要发送的消息，进行邮件的相应操作。是不是很方便，这就是消息队列。你不用知道具体的服务在哪，如何调用。你要做的只是将该发送的消息，向你们约定好的地址（信箱）进行发送，你的任务就完成了。对应的服务自然能监听到你发送的消息，进行后续的操作。这是消息队列最大的特点，将同步操作转为异步处理，将多服务共同操作转为职责单一的单服务操作，做到了服务间的解耦。好，万无一失了吗？  
不久的一天，你发现所有业务都替换了邮件发送的方式，统一使用了消息队列来进行发送。这下仅仅一个邮件服务模块，难以承受业务方源源不断的消息，大量的消息堆积在了队列中，队列服务器也面临瘫痪。这就需要更多的消费者（邮件服务）来共同处理队列中的消息，这就是我们今天的主角-分布式消息处理。  
###2 Kafka简介  
Kafka是由Linkedin开发的消息队列，使用Scala语言编写，之后成为Apache项目的一部分。Kafka是一个分布式的、多分区的、多消费者、多冗余备份的、持久性的消息系统。它主要用于处理活跃的流式数据。Kafka设计的初衷是希望作为一个统一的信息收集平台，能够实时的收集反馈信息，并能够支撑较大的数据量，且具备良好的容错能力。我们首先看看Kafka提供的一些非常棒的特性。  
#### 消息持久化  
Kafka采用时间复杂度O(1)的磁盘结构顺序存储数据，即按顺序访问磁盘。这很多时候比随机的内存访问快的多，而内存作为磁盘的缓存。Kafka直接将数据以追加的形式写入到日志文件中，以没有容量限制（相对于内存来说）的硬盘空间建立消息系统。这样读操作不会阻塞写操作与其他操作，因为读和写都是追加的形式，都是顺序的，不会乱。  
#### 高吞吐量  
Kafka支持每秒百万级别的消息，它的延迟最低只有几毫秒。Kafka的高吞吐量体现在读写上，分布式并发的读和写都非常快，写的性能体现在以o(1)的时间复杂度进行顺序写入。读的性能体现在以o(1)的时间复杂度进行顺序读取。Kafka对topic进行partition分区，consume group中的consume线程可以以很高能性能进行顺序读。  
#### 易扩展  
Kafka是分布式的消息系统，它借助Zookeeper来实现集群管理。当集群性能不够时，可以简单的新增机器，配置好，集群就可以自动感知，而无需停机。  
#### 高容错
在Kafka中消息按topic存储，每一个topic又可以分多个partition。kafka中，冗余备份的策略是基于partition，而不是topic。Kafka将每个partition数据复制到多个server上，任何一个partition有一个leader和多个follower。多副本的冗余存储为Kafka提供了高容错性，它允许集群中有节点失败（若副本数量为n，则允许n-1个节点失败）。  
#### 实时性  
进入到kafka的消息能够立刻被消费者消费。当生产者将消息发送到Kafka后，就会去立刻通知ZooKeeper，Zookeeper会watch到相关的动作，当watch到相关的数据变化后，会通知消费者去消费消息。消费者是主动去Pull（拉）kafka中的消息，这样可以降低Broker的压力，因为Broker中的消息是无状态的，Broker也不知道哪个消息是可以消费的。当消费者消费了一条消息后，也必须要去通知ZooKeeper。zookeeper会记录下消费的数据，这样当系统出现问题后就可以还原，可以知道哪些消息已经被消费了。  
###3 Kafka原理  
Kafka整体架构如图4-22所示。  
![](/assets/Kafka原理.png)  
图4-22
在kafka中，发送消息者称为producer，而消息拉取者称为consumer。通常consumer是被定义在consumer group中。kafka通过Zookeeper管理集群。同一个消息可以被多个consumer group拉取处理，但是在一个consumer group里只能有一个consumer处理该条消息。同一个group中的consumer之间是竞争互斥的关系。
kafka集群由多个实例组成，每个节点称为Broker。一个Topic可以被划分为多个Partition。每个Partition可以有多个副本。Kafka会尽量将同一个partition的不同副本均匀分布到不同的broker。如图4-23。  
![](/assets/partition.png)
图4-23。  
图中一共有三个broker，两个topic。topic1有两个partition，topic2有一个partition，各有3个副本。  
每一组partition有一个leaer，其余为follower。leader负责读写数据，follower只负责与leader同步数据。  
Partition内消息顺序存储，写入新消息采用追加的方式，消费消息采用FIFO的方式顺序拉取消息。Kafka只保证同一个分区内有序，不保证Topic整体（多个分区之间）有序。如图4-24。  
![](/assets/seq.png)
图4-24。  
下面介绍几个Kafka中的核心概念。  
#### Broker  
启动kafka的一个实例就是一个broker，一个kafka集群可以启动多个broker。通常，一个broker就是一台server。  
#### Topic  
kafka中同一种类型数据集的名称，相当于数据库中的表。producer将同一类型的数据写入同一个topic下，consumer从同一topic消费同一类型的数据。  
#### Partition  
一个topic可以设置多个分区，相当于把一个数据集分成多份，分别放到不同的分区中存储，一个topic可以有一个或者多个分区，分区内消息有序。  
#### Replication  
partition的副本。一个partition可以设置一个或者多个副本，副本主要保证系统能够持续不丢失的对外提供服务，提高系统的容错能力。在0.8以前是没有Replication的，一旦某台broker宕机，其上partition数据便丢失。  
#### Producer  
消息生产者，负责向kafka中发布消息。producer端发送的message必须指定是发送到哪个topic，但是不需要指定topic下的哪个partition，因为kafka会把收到的message进行load balance，均匀的分布在这个topic下的不同的partition上。kafka集群中的任何一个broker,都可以向producer提供metadata信息，这些metadata中包含集群中存活的brokers列表和partitions leader列表等信息。当producer获取到metadata信息之后, producer将会和topic下所有partition leader保持socket连接;消息由producer直接通过socket发送到broker,中间不会经过任何"路由层"。  
#### Consumer Group  
消费者所属组，一个CG可以包含一个或多个consumer，当一个topic被一个CG消费的时候，CG内只能有一个consumer消费同一条消息，不会出现同一个CG多个consumer同时消费一条消息造成一个消息被一个CG消费多次的情况。一个CG内的consumer对消息的处理逻辑是相同的，不同的CG的consumer对消息的处理逻辑不同。  
#### Consumer  
消息消费者，consumer从kafka指定的主题中拉取消息。每个consumer客户端被创建时，会向zookeeper注册自己的信息，此作用主要是为"负载均衡。一个group中的多个consumer可以交错的消费一个topic的所有partitions。简而言之,保证此topic的所有partitions都能被此group所消费，且消费时为了性能考虑，让partition相对均衡的分散到每个consumer上。一般，consumer数量与topic下的partition数量相等会达到最大效率。  
#### Zookeeper  
Zookeeper在kafka集群中主要用于协调管理，kafka将元数据信息保存在Zookeeper中，通过Zookeeper管理维护整个kafka集群的动态扩展、各个broker负载均衡、Partition Leader选举等。  
###4 Kafka存储
在Kafka文件存储中，同一个topic下有多个不同partition，每个partition为一个目录，partiton命名规则为topic名称+有序序号，第一个partiton序号从0开始，序号最大值为partitions数量减1。每个partition目录下是segment（段文件），segment是kafka中最小数据存储单位。一个partition包含多个segment文件，每个segment以message在partition中的起始偏移量命名，以log结尾。如图4-25。  
![](/assets/log文件.png)  
图4-25。  
比如有100条消息，它们的offset是从0到99。假设将数据文件分成5段，第一段为0-19，第二段为20-39，以此类推，每段放在一个segment里面，以该段中最小的offset命名。这样在查找指定offset的消息的时候，用二分查找就可以定位到该消息在哪个段中。
kafka为了提高写入、查询速度，在partition文件夹下每一个segment log文件都有相同的索引文件，在kafka0.10以后的版本中会存在两个索引文件。一个用offset做名字以index结尾的索引文件，我们称为偏移量索引文件。一个是以消息写入的时间戳作为名字以timeindex结尾的索引文件，我们称为时间戳索引文件，如图4-26。  
![](/assets/index.png)
图4-26
我们简单介绍一下偏移量索引文件，时间戳索引文件同理类似。
偏移量索引文件以偏移量作为名称，index为后缀。采用稀疏存储方式，每隔一定字节的数据建立一条索引（这样的目的是为了减少索引文件的大小）。索引文件的内容格式为offset，position（物理存储位置）。
图

时间戳索引文件：
以时间戳作为名称，以timeindex为后缀。时间戳是由producer来决定，有两种时间，一种是消息创建时间的时间戳，一种是消息写入队列时间的时间戳。
索引内容格式：timestamp，offset
采用稀疏存储方式
通过log.index.interval.bytes设置索引跨度

###4 Kafka高可用
kafka高可用：
1、多分区、多副本
2、kafka Controller选举
从集群中的broker选举出一个Broker作为Controller控制节点。控制节点负责整个集群的管理，如Broker管理、Topic管理、Partition Leader选举等。
选举过程通过向Zookeeper创建临时znode实现，未被选中的Broker监听Controller的znode，等待下次选举。
3、kafka Partition Leader选举
Controller负责分区Leader选举
ISR列表：Follower批量从Leader拖取数据，Leader跟踪保持同步的follower列表ISR（In Sync Replica），ISR作为下次选主的候选列表。Follower心跳超时或者消息落后太多，将被移除出ISR
Leader失败后，从ISR列表中选择一个Follower作为新的Leader。


