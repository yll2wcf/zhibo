##4.3.3 分布式消息队列与Kafka  
kafka基本原理
简介
Kafka是由Linkedin开发的消息队列，使用Scala语言编写。
分布式、多分区、多副本、基于发布/订阅的消息系统
按topic存储，为了提高吞吐率，每个topic分成多个分区，每个分区可以有副本。生产者将消息发布到kafka中，消费者订阅主题。
Kafka设计的初衷是希望作为一个统一的信息收集平台，能够实时的收集反馈信息，并能够支撑较大的数据量，且具备良好的容错能力。

特性
消息持久化：采用时间复杂度O(1)的磁盘结构顺序存储
高吞吐量：支持每秒百万级别的消息
易扩展：新增机器，集群无需停机，自动感知
高容错：通过多分区，多副本的冗余存储提供高容错性
多客户端支持：JAVA\PHP\PYTHON等
实时性：进入到kafka的消息能够立刻被消费者消费

消息队列的作用
消息的临时存储区域、提供数据的缓冲，保证系统的稳定性
应用程序解耦并行处理。传统的消息处理是采用串行的方式，
顺序保证 一个topic内不一定有序，分区内是有序的。
高吞吐率
高容错、高可用
可扩展
峰值处理


原理图
在kafka中，发送消息者称为Producer，而消息拉取者称为Consumer。通常，consumer是被定义在Consumer Group里
kafka通过Zookeeper管理集群。同一个消息可以被多个Consumer Group拉取处理，但是在一个Consumer Group里只能有一个Consumer处理该条消息。Consumer之间是竞争互斥的关系。
kafka集群由多个实例组成，每个节点称为Broker，对消息保存时根据Topic进行归类。
一个Topic可以被划分为多个Partition。每个Partition可以有多个副本。
图
每一组partition有一个leaer，其余为follower。leader负责读写数据，follower只负责与leader同步数据。

Partition内消息顺序存储，写入新消息采用追加的方式，消费消息采用FIFO的方式顺序拉取消息。
一个Topic可以有多个分区，Kafka只保证同一个分区内有序，不保证Topic整体（多个分区之间）有序。
图

Consumer Group（CG），为了加快读取速度，多个consumer可以划分为一个组，并行消费一个Topic。一个Topic可以由多个CG订阅，多个CG之间是平等的，同一个CG内可以有一个或多个consumer，同一个CG内的consumer之间是竞争关系，一个消息在一个CG内只能被一个consumer消费。

一个CG内的consumer对消息的处理逻辑是相同的，不同的CG的consumer对消息的处理逻辑不同。


kafka集群部署