---
title: redis
date: 2020-03-31 19:57:53
tags:
- linux
- 学习
- 数据库
categories : "数据库"
---

> - redis学习笔记
> - 主要记载redis使用、底层知识
> - 串写Zookeeper和kafka
> - 技术易于人的使用，理论却很复杂

<!--more-->

# 传统数据库data page
传统数据库的data page一般大小为4K，和底层磁盘一次读取4K相同。因此data page可以大于4K，但是小于4K，会造成磁盘读取4K数据的浪费
- 数据库索引在内存当中只存B+树（树干），而命中索引后，会读取对应的data page

# redis支持的类型
redis支持的字符串（二进制安全，字节数组存储），散列，列表，集合，有序集合，这里指的是value支持，而不是key。而memcached的value则没有类型的概念。

# 基础help
基础help命令主要通过两个维度查询
```
help @类名
help 命令
```

# bitmap
在大批量数据时，可以通过bitmap来节省空间，如统计两千万用户在某一天是否存在登录，offset则为用户ID，或者是统计用户登录的区间，每一天以一个bit代替

## 布隆过滤器
使用bitmap有可能造成数据冲突，所以我们可以定义K个哈希函数，将一个KEY，多次哈希之后，再次映射到bitmap当中，这样可以保证，如果得到结果为不存在一定不会误判。但是会带来一个问题，假设这个数据不存在，但是被判断为了存在，从而造成数据丢失。
- 我们可以通过调整哈希函数的个数，位图大小，要存储的数字比例，可以将这种误判尽可能降到最低。

# list
list有序，但是针对的是插入有序，而非数据有序

# set和sorted_set
## set
set是无序的，且支持集合的交并差操作，允许随机抽取数据，如：SRANDMEMBER key [count]，后面的count如果是正数，则一定取出对应个数量的元素，且保证唯一，但是如果元素不够的话，则会返回整个集合的所有元素。后面的count如果是负数，则数据元素不保证唯一，取出即可。

## sorted_set
如何排序是一个问题，假设有苹果、香蕉、梨，他们可以通过含糖量，大小，价格，热度等等，即排序规则需要自己指定
- 如果排序的值都为1，则会按照名称字典序排序。
- 默认是从小到大排序（左小右大）
- 同样具备集合操作

### 跳跃表（skiplist）
跳跃表的查找类似于二分查找，但是如果多次插入数据，如果不更新上层索引，则会出现两个索引节点当中，存在的数据节点非常多。因此我们可以定义一个随机函数，来决定将这个节点插入到哪几级索引当中，比如随机函数生成了K，我们就将节点添加到第一级到第K级索引中。

# 管道
服务器会按顺序处理请求，因此可以一次性发送多个命令，使用管道传输，降低RTT开销
```
(printf "PING\r\nPING\r\nPING\r\n"; sleep 1) | nc localhost 6379
```

# 发布订阅
SUBSCRIBE来监听一个频道，PUBLISH可以推送数据到一个频道。并且所有监听频道的客户端都可以收到数据。

# 事务
redis事务不像mysql那么完整，谁的命令exec先到达，就先执行谁的事务。因此没办法确定一个值是否一定可以正确获取。
- watch指令可以在开启事务前，进行监听key的value。如果开启事务后，监听的值被更改了，事务不会执行

# 模块
redis4.0之后加入了模块，因此可以导入第三方模块（如布隆过滤器）

## 布隆过滤器（redis4.0之后支持）
主要用于解决缓存穿透的问题，如果不存在，直接结束，不允许请求数据库
- [布隆过滤器](https://oss.redislabs.com/redisbloom/)

# LRU缓存
- 业务逻辑：key的有效期
- 业务运转：内存是有限的，随着访问的变化，应该淘汰冷数据
## 回收策略（摘自文档）
[lru-cache](http://redis.cn/topics/lru-cache.html)
- noeviction:返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）
- （常用）allkeys-lru: 尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。
- （常用）volatile-lru: 尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放。
- allkeys-random: 回收随机的键使得新添加的数据有空间存放。
- volatile-random: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。
- volatile-ttl: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。

可以看出主要分为LRU和随机淘汰，但是每个算法中间可以对应全部键以及有剩余时间的键

## ttl时间
如果一个key设置了过期时间，读不会让时间变更，但是如果重新写，会造成时间丢失，即被重置

# 如何淘汰过期的key
两种方式：主动、被动
- 被动：客户端访问发现过期了，剔除
- 主动： 每10秒测试随机的20个keys进行检测剔除
- [expire key seconds](http://redis.cn/commands/expire.html)

# 持久化
## 管道
- 衔接，前一个命令的输出作为后一个命令的输入
- 管道会触发子进程创建

echo $$可以输出当前进程的pid，echo $$ | more结果不变，但echo $BASHPID | more，则值会不同。因此可以得出结论|的运算优先级大于普通变量的优先级，小于系统变量的优先级

- 引出linux环境中，子进程修改变量不会影响到父进程（fork后开启双进程）
## 引出问题
1. 如果原来redis内存10G，复制后会变成20G吗
2. 创建子进程的速度应该时什么速度

## RDB（全量复制）
时点性，因此为了保证时点性（某个时间点的数据），所以使用fork出一个子进程，fork依赖写时复制，创建进程变快了，一旦新数据被写入，不会影响到子进程的拷贝。

- save命令（前台阻塞）
- bgsave命令 （后台触发）
- 配置文件中编写bgsave规则
```
save <second> <changes>
#默认
save 900 1     #900秒，1个key修改
save 300 10    #300秒，10个key修改
save 60 10000  #60秒，10000个key修改
```

弊端：不支持拉链，永远只有一个dump.rdb，因此需要运维写脚本去备份前一个时间点的rdb文件

## AOF（append only file）
redis的写操作直接记录到文件中。因此和rdb相比，丢失数据相对较少，但是效率较低。

- rdb和aof可以同时开启，如果开启了aof只会用aof恢复
- 4.0之前，日志会删除抵消的命令，合并重复的命令，最终也是一个纯指令的日志文件（重写）
- 4.0版本之后，AOF中包含rdb全量（存放老的数据），增加记录新的写操作（重写）

写操作会触发IO，但是对于内存数据库来说会拖慢效率。因此aof有三个级别
1. NO，redis不调用flush，内核buffer满了，就写入到硬盘。因此可能丢失一个buffer的数据
2. always，最可靠的模式，每笔都调用flush
3. everysec，每秒flush一次

# redis集群
## 问题
1. 单点故障（单个机器挂掉）
2. 容量有限
3. 连接数压力（socket io、cpu压力）

## akf（一边多）
1. 沿着X轴扩展redis台数，读写分离。基于X轴，基本是全量镜像，即所有redis保存一样的数据
```
        |          x轴         |
client-> redis-redis-redis-redis
```
- 主备模式，因此如何保证redis数据的同步

2. 沿着y轴扩展，按照不同功能扩展redis（每个redis处理不同的业务）。在mysql当中就是分库
```
redis
  |
redis
  |
redis
```

3. 沿着z轴扩展，基于一个业务分成多个redis，保证一个规则，如用户id，1到一亿在第一个redis，一亿之后在第二个redis
- 优先级，逻辑再拆分

## 数据一致性问题
1. 所有节点阻塞，直到所有数据全部一致（强一致性）。但是会破坏可用性，如服务不可用，网络延时

强一致性必须通过同步方式，而通过异步方式，即写redis直接向客户端返回ok，再将自己的数据同步到其他redis节点，有几率造成**数据丢失一部分**。因此有一种解决方案如下：
```
                                              |->redis
client -> set k1 123 -> redis ->（同步） kafka |
                                              |->redis
kafka必须要保证可靠，集群，响应速度够快
```
但是以上这种方案是最终一致性，因此中途取数据的时候有可能取到不一样的数据

## 监控
一般来说，监控的机子数量为奇数台。原因时和偶数台相比，比如5台和6台相比，6台需要4台才能投票出被监控的机子是否挂掉，而5台只需要三台就可以投票，并且他们允许失误的机子都为2台

## 主从主备
### 主从（常用）
客户端可以访问主redis，副redis也可以访问。

### 主备
client明确访问第一个redis，而后面的redis只在主redis挂掉之后顶上去。

### 带来的问题
主又是一个单点，因此我们需要对主做高可用

#### 脑裂
当使用主备机制时，如果因为网络原因，造成主master假死，而备slave顶上来，但后续master恢复正常之后，系统存在两个“主”，就可以看成时发生了脑裂的问题。

## 主从复制（默认异步）
- 5.0之前使用SLAVEOF host port，5.0之后为replicaof，代表从机追随谁。默认情况下，redis从机禁止写入
- replicaof no one会关闭追随
- 一旦开启aof，会进行全量同步
- 只开启rdb，会记录replicid，进行增量同步
- 同步有两种方式，一种是主机先IO写RDB，再网络IO传过去。另一种是直接网络IO传过去。

## 哨兵(Sentinel)
启动哨兵时，只需要给出主机的ip，port以及投票的权重值（分区容忍性）。一套哨兵可以监控多个主从集群。
```
PORT 26379
sentinel monitor mymaster 127.0.0.1 6379    2
                 #哨兵名称 #ip      #端口   #投票权重值
```
哨兵主要通过主redis的发布订阅来发现其他哨兵或者其他从机

## cluster
[redis集群教程](http://redis.cn/topics/cluster-tutorial.html)
在业务层面上，没有所谓的主，即无主模型。客户端想连谁连谁，在redis当中保存一个mapping，如果不在当前的redis，则返回重定向，因此客户端去访问另外一个redis

redis没有使用一致性哈希，而是使用了哈希槽的概念。且每个主机都保存路由表。

- 使用了集群之后，就无法使用watch或者multi

# CAP定义
## 一致性
所有操作执行完毕，客户端收到返回请求。所有节点在同一时间，数据完全一致

## 可用性
服务一直是在正常的响应时间

## 分区容错性
在分布式系统当中，如果某节点出现故障，对外来看服务依然可用。

## 为什么三者不能同时满足
- 假设我们同时满足CA，即强一致性和可用性，就意味着服务不能被中断，必须牺牲分区，有分区就有可能出现通信失败，造成不可用
- 如果满足CP，即要满足强一致性和分区，那就必须同步操作，若网络延时，造成同步阻塞，则可用性不满足
- 如果满足AP，要服务可用，且各分区允许容错，那么就没办法保证强一致性

一般来说银行一定要满足C，其他行业大多数是先满足P，然后再CA间进行权衡

# 缓存的四个问题
## 击穿
前提：高并发。大量数据在缓存失效后，直接打到数据库连接。因此解决方案如下：
```
1. get key
2. setnx //如果没拿到key，去抢锁
3. 拿到锁去访问db，重新创建key。没拿到锁就sleep
4. 回到1
```
- 但是如果在第二步中，拿到锁的线程挂了，就会造成死锁。因此我们可以设置锁的过期时间（锁的过期又是另外一个问题，所以可以用多线程/守护线程来监控锁，延长过期时间）。

## 雪崩
同一时间，大量的key失效。
- 解决方案：随机过期时间

## 穿透
客户端查询数据库根本不存在的数据，导致跳过缓存，直接打到数据库上。
- 解决方案：布隆过滤器。但是布隆过滤器，只能增加不能删除。

# redis分布式锁
redis虽然也可做分布式锁，但是会带来一些过期时间问题。因此推荐使用zookeeper来做分布式锁。

# redis api注意事项
redis本身不保证线程安全，即如果client端使用了多线程，就需要自己保证线程安全，因此有一个解决做法是维护一个pool，每次从pool取出一个连接来处理。

# zookeeper（分布式协调服务）
zookeeper和redis一样，也基于内存。redis完成分布式协调很难，因此zookeeper可以简单的解决分布式锁问题（众多功能之一，session关掉，临时节点消失）

zookeeper是一个目录树结构，并且每个节点可以存储数据（1MB官方上限、二进制安全）。节点分为持久节点、临时节点（两者可以为序列节点）。

- 二进制安全：客户端推过来什么字节数组，就保存什么字节。

zookeeper主从模型，保证leader在的时候，满足:
- 顺序一致性（客户端更新将按顺序发送应用）
- 原子性 
- 单系统映像 （无论连到哪个服务器，客户端都将看到相同的服务视图，连session都是统一的）
- 可靠性 （一旦应用了更新，客户端将覆盖更新）
- 及时性

zookeeper集群需要人为规划，需要在配置文件当中写下：
```
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888
server.4=node04:2888:3888
//票数为行数/2+1
//3888代表leader挂掉后，或者不存在，四个服务会在3888端口进行投票，选出leader后回2888通信。
```

## get命令
- cZxid为创建的事务id
- ctime创建时间
- mZxid最近修改的id
- mtime修改时间
- pZxid最近创建的节点事务id

## 使用场景
- 统一配置管理 -> 1M数据存储配置
- 分组管理 -> path结构
- 统一命名 -> -s参数 sequential序列
- 分布式同步 -> 临时节点

不推荐往zookeeper一直写入数据，读就可以了。

## 角色
1. leader
2. follower
3. observer 放大读查询能力

## 读写分离
只有follower才能选举

## paxos
基于消息传递的一致性算法。过半通过，两阶段提交。
- 前提：没有拜占庭将军问题（网络当中所有线路通信是健康的，节点与节点通信是可信的）
- 参考：[paxos](https://www.douban.com/note/208430424/)

## ZAB
zookeeper的原子广播，作用在可用状态，有leadler时。可以近似看成paxos协议的一个精简版。
- 原子：要么成功、失败，没有中间状态
- 广播：分布式多节点。广播不代表全部都知道

Leader为每个Follower维护了一个队列，用于发送log，以及write（一阶段log和二阶段提交）

## watch
可以观察到子节点是否有变化（节点是否有产生事件：create,delete,change,children），如果有变化，调用callback

# 分布式协调场景
## 分布式配置发现
机器watch zoolkeeper里面的配置项，如果变更会发生回调。

## 分布式锁
1. 争抢锁，只有一个人能获得
2. 获得锁的人出问题，用临时节点解决
3. 获得锁的人处理完毕，释放锁
4. 锁被释放，如何被人知道。
- 主动查询：心跳，但是有弊端，延时，压力，如1000台机器。
- watch： 观察一个节点，等待通知。可以解决延迟问题，但是会引出泛洪效应。
- sequence-watch：watch前一个sequence，最小的获得锁。在获得到锁之后，可以setData,设置一个标记，可以通过判断标记再次拿到锁，即可重入锁。

# mmap映射
传统读取文件，需要经历一个过程：read->内核->文件，这就意味着，文件先从磁盘拷贝到内核，再从内核拷贝到应用层。

使用mmap映射，可以在内存当中开辟一个空间，内核和应用层可以直接访问，而不需要一次拷贝的过程。

## kafka
kafka持久化的时候，可以通过mmap映射空间,这样无需走先拷贝到内核空间，内核空间再拷贝到磁盘这个过程。

### sendfile(out_fd,in_fd)零拷贝
内核直接从文件当中，将数据发送给socket，而不需要复制到应用层
- sendfile 是将 in_fd 的内容发送到out_fd。而in_fd不能是socket，也就是只能文件句柄。