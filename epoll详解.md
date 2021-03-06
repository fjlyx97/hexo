---
title: epoll详解
date: 2020-03-09 22:55:34
tags:
- 编程
- 学习
- 网络编程
categories : "socket编程"
---

> - IO多路复用
> - epoll原理
> - reactor模式

<!--more-->

# 为什么recv等阻塞不消耗cpu资源
操作系统将进程分为多种状态，并且在内核空间维护对应的工作队列。当调用**socket**函数时，操作系统会在底层创建一个文件对象（fd），这个对象包括了**发送缓冲区**，**接收缓冲区**，**等待队列**等成员。当调用**recv**函数时，操作系统会将进程状态修改为等待，并将其加入到对应**fd等待队列**。因此进程被阻塞，不消耗cpu资源。

## 唤醒
当socket接受到数据时，操作系统会将对应fd中，等待队列里所有的进程放回工作队列，与此同时缓冲区有了数据，进程可以进行读取

# 阻塞非阻塞/同步异步
## 阻塞I/O模型
套接字默认都是阻塞的，如调用了recv函数，如果没有数据就会一直等待，知道有数据后，将数据从内核拷贝到应用缓冲区

## 非阻塞I/O模型
非阻塞则是类似一种轮询的方式，应用进程调用recv一次次去询问内核，如果无数据就继续往下走，有数据则进行数据拷贝。

## I/O多路复用
严格来说，I/O多路复用函数也是阻塞的，但是它阻塞的位置是在select或者epoll这样的系统调用上，而不知阻塞在I/O系统调用如recvfrom上。

## 异步I/O
不同于同步非阻塞的轮询，系统通知内核启动某个操作，内核完成后返回通知给进程，异步不阻塞。

# 多路复用
在这里多路可以看成**多个socket**，而复用可以看成**复用一个线程**。

传统方案当中，当服务端accept到一个socket时，会开启一个线程去处理对应的数据，但是线程是有限的，并且线程上下文切换会产生大量的开销，因此并不推荐使用。

因此需要**一个线程管理多个socket**，epoll则是其中一个高效的方法。既然是高效的方法，就有着反面教材，其中包括**select，poll**
## select
select是最早的IO复用模型，用结构体fd_set来告诉内核监听多个文件描述符。但是它存在着很多的问题，比如在windows当中，如果不做修改，最大可以监听1024个文件描述符。查询数据时，每次需要将fd_set拷贝到内核态，造成无用开销。

每次调用select，都需要将对应的fd从用户态拷贝到内核态，造成大量的无用复制，并且要得到哪些socket有数据，需要轮询一边数组

## poll
poll模型和select模型非常相似，但是它的文件描述符数量不受限制，因为它是使用**链表存储**。这点小优化并没有解决本质上的问题，就是查询数据需要轮询数组所有的元素，因此epoll应运而生

## epoll
select低效的原因在于不知道那些socket拥有数据，只能遍历所有fd。因此epoll做了改进，它在内核维护一个就绪列表，由**双向链表**组成，当epoll_wait唤醒时，只需获取双向链表的内容，就可以知道哪些socket有数据。

### 创建epoll
类似与socket的创建，epoll创建时会在内核创建一个eventpoll对象，它同样拥有一个等待队列。当进程调用wait时，会被加入等待队列，直到有数据时才会被唤醒。

### 就绪列表为什么时双向链表
就绪列表本质上是为了储存哪些socket发生了数据变化，因此需要大量的增加或者删除，因此双向链表最为符合。

### 索引结构为什么是红黑树，而不是hashmap
epoll底层用红黑树来管理文件描述符。红黑树兼顾了插入，删除，查找，且时间复杂度都不错，如果使用hashmap的话，可能会占用大量的空间，虽然提升的一定的效率，但是在内核这种空间珍贵的地方，并不适用。

如果我们使用epoll_ctl添加节点，第一个使用的就是红黑树的查找功能，根据网上资料，红黑树中排序的依据是通过**file的地址大小**（不确定），如果已经存在则直接返回。否则再根据相应的命令，对节点进行插入。再插入的过程中，同时注册**回调函数**，内核在检测到fd有数据时，会调用回调函数，将fd直接加入等待队列。

### 水平触发和边缘触发
这个类似与数字电路中的概念，即低电平，高电平，电平边缘

#### ET边缘触发
1. 缓冲区由空变为不空的时候。
2. 当有新数据到达时

由于ET模式只会触发一次，所以，需要单独编写循环，确保所有数据读取结束，并且需要将套接字设置为非阻塞，防止因为被阻塞导致其他线程饿死

#### LT水平触发
1. 对于读操作，只要缓冲内容不为空，LT模式返回读就绪。
2. 对于写操作，只要缓冲区还不满，LT模式会返回写就绪。

因此，只要有数据，如果自己不去触发读写，epoll_wait会一直收到提醒

# reactor设计模式
传统网络编程就是服务器一个while循环，不断监听端口是否有套接字连接，如果有的话就处理一下。但是这种方法效率极低，且容易阻塞住后面的线程，因此多线程被提出用来优化

多线程处理就是很经典的connection per thread，即每个连接用一个线程处理，早期的tomcat就是这样实现的。虽然这样可以极大的提高吞吐量，但是需要大量的系统资源，如果连接数太高将系统资源数耗尽，则无法创建新线程。
