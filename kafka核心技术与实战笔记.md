---
title: kafka核心技术与实战笔记
date: 2020-05-07 10:25:09
tags:
- 编程
- 学习
- 消息队列
categories : "消息队列"
---

> - 阅读kafka核心技术与实战
> - 阅读Apache Kafka实战
> - 记录学习kafka的过程

<!--more-->

# 入门术语
- 消息：Record。Kafka 是消息引擎嘛，这里的消息就是指 Kafka 处理的主要对象。
- 主题：Topic。主题是承载消息的逻辑容器，在实际使用中多用来区分具体的业务。（给队列起名字，生产者知道往哪里丢数据，相当于数据库表的概念）
- 分区：Partition。一个有序不变的消息序列。每个**topic(主题)**下可以有多个分区。
- 消息位移：Offset。表示分区中每条消息的位置信息，是一个单调递增且不变的值。（消息位移**恒确定**，不会超过最新一条消息的位移）
- 副本：Replica。Kafka 中同一条消息能够被拷贝到多个地方以提供数据冗余，这些地方就是所谓的副本。副本还分为领导者副本和追随者副本，各自有不同的角色划分。副本是在分区层级下的，即每个分区可配置多个副本实现高可用。
- 生产者：Producer。向主题发布新消息的应用程序。
- 消费者：Consumer。从主题订阅新消息的应用程序。
- 消费者位移：Consumer Offset。表征消费者消费进度，每个消费者都有自己的消费者位移。（动态变化）
- 消费者组：Consumer Group。多个消费者实例共同组成的一个组，同时消费多个分区以实现高吞吐。（多个消费者，消费一个分区）
- 重平衡：Rebalance。消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。Rebalance 是 Kafka 消费者端实现高可用的重要手段。
- 缓存代理：Broker，Kafka集群中的一台或多台服务器统称broker。集群就是**多台Broker**

# 为什么kafka适合部署在linux
主流的 I/O 模型通常有 5 种类型：阻塞式 I/O、非阻塞式 I/O、I/O 多路复用、信号驱动 I/O 和异步 I/O。kafka使用了java的selector，其底层在linux上为epoll而在windows上为select，因此明显linux更优。

网络模型同样有差异，linux当中拥有zero copy技术，可以避免磁盘和网络间的性能损耗。

# 磁盘问题
Kafka大量使用磁盘，但是基于**顺序读写**，因此SSD并没有太大的性价比。

- 在顺序读写上，磁盘的IO速度甚至可以匹敌内存的随机访问速度

规划磁盘容量需要考虑以下几点：
1. 新增消息数
2. 消息保存时间
3. 消息大小
4. 备份数
5. 压缩率

# Broker重要参数
## 存储信息
- log.dirs：非常重要，无默认值，需要自己制定，为每个Broker指定日志文件位置。

## Zookeeper
- zookeeper.connect，设置zookeeper集群，可以通过proxy来达到只设置一个的目的。

***

# Apache kafka 实战
以下为阅读Apache kafka 实战的笔记

# kafka消费端如何做到高吞吐量、低延时
Kafka把消息顺序写入页缓存，那么同样，读缓存也会从OS中页缓存读取，并直接转发到socket上，不经过应用层，这就是零拷贝。
1. 大量使用页缓存
2. 不直接参与物理IO操作，交给操作系统
3. 顺序读写
4. 使用零拷贝

# Kafka topic结构
出于性能的考量，Kafka并不是用topic-message两级结构，而是使用topic-partition-message三级结构来负载。因此，一个topic可以看成由多个partition来负载。

- 用户对partition唯一能做的，就是在消息队列尾部追加写入消息。

# Partition
分区。topic的消息被分为多个分区，对用到各个服务器。单个分区中消息是有序的，但是多个分区，消息不一定是有序的。一般情况下，一个Topic的分区数量是Broker数量的**整数倍**。这样可以将分区均匀分布到各个Broker上。

- 保存位置为：log.dirs的路径
- 有一个特殊的Partition：consumer_offset

# Segment
将分区分割为若干个段，每个segment文件最大大小相等。因为如果Partition不分区，存储的内容多了之后，不好管理，容易出问题

# Consumer Group
在同一个消费者组内，他们共享同一个公共ID，即GroutId，他们平均消费的对象针对的**不是消息**，是**分区**。不允许同一个组里两个消费者，消费同一个分区。

# Kafka message
要定位到一条消息，我们必须要三个数据：topic,partition,offset，只要有了这三个数据，就可以定位到任意一条数据，因此一条消息本质上可以看成是一条**三元组**

# leader和follower(针对Replica)
要保证数据不丢失，就需要备份数据（replica），在kafka中分为leader和follower。这种做法和传统mysql主从的做法不同的是，只有leader对外服务，follower只能从leader获取数据，一旦leader挂了，follower才会顶上（主备）。

- kafka保证replica不会在同一个broker上（不然机器挂了和没备份一样）

# 分区机制
对于有Key的消息而言，Java版本producer自带的partitioner会根据**murmur2**算法，计算hash，接着对总分区取模，确定要发送到的目标分区号。但是一般来说，允许自定义处理机制，这样就可以满足如：相同key存储到最后一个分区，否则随机存储到除最后一个分区以外的地方。

# ISR(In-sync replicas)
副本同步列表，最初的时候副本全都是AR(Assigned Replicas)，如果断线了会变成OSR(outof-sync replicas)，AR=ISR+OSR。
- ISR列表由

# 创建一个消费者
## bootstrap.servers
这个参数主要是为了找到Kafka环境，只要写了任意一个IP，kafka总能找到全部broker,但是多写几个可以防止单点kafka挂掉，找不到其他Broker的情况

## groutid
指定消费者组的ID，是一个字符串

# Reblance
## 触发条件
1. 组成员发生变更，如：新消费者入组，已有消费者离组（或崩溃）
2. topic数量发生变更
3. topic分区数发生变更

### Range
基于范围的思想，将单个Topic分区顺序排列，将分区按照固定大小的**分区段**，一次分配给每个consumer

### Round-robin
topic分区全部摆开，轮询式分给每个consumer

### Sticky
最新发布的思想，采用了**有粘性**策略，

# Broker Controller
Kafka集群中有多个broker，有一个会被选举为Controller，，负责控制集群中的分区和副本（集群中的老大）
- topic的分区变化时，由Controller负责管理Partition与消费者的分配关系，即Rebalance

# 脑裂
一个集群，旧master假死，新master被选举出来，造成两个master的情况。解决方法，新master上任后，延时开始工作，直到旧master变为replicator

# HW和LEO（重点内容）
## HW
- HW,HighWaterMark高水位

HW代表目前可以消费到的最高位置，通常在同步完一个LEO之后，才会更新HW

## LEO
- Log end offset

LEO是各自分区的消息offset，但是各个分区没有完全同步LEO之前，是无法消费的。