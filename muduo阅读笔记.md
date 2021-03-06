---
title: muduo阅读笔记
date: 2020-04-27 10:52:16
tags:
- 编程
- 学习
- C家族
- 多线程
categories : "C家族"
---

> - 阅读Linux多线程服务端编程-使用muduo C++库
> - 记录思维、语法、坑点

<!--more-->

# 多线程销毁对象
Mutex只可以保证函数一个接一个执行，但是如果有两个对象，一个对象正在执行类成员函数，而另外一个对象执行了析构函数。这样则有可能造成锁被销毁，永远无法获得。
- 如果一个函数如果要锁住相同类型多个对象，需要比较锁地址，每次锁住最小的的mutex

# 多线程析构
在编写长期运行的多线程服务程序时，可以不必追求安全退出。可以让服务进入不可用的状态，然后直接杀掉杀进程即可。

# 判断一个指针是否合法
在java当中，如果一个对象(reference)不为NULL，则一定指向一个有效对象。C++不应该使用原始指针进行保存，而是需要封装成一个类，可以用isAlive来判断类是否存活的类。

销毁对象时，最好在别的线程看不见对象时偷偷的做。（别人用不到的东西一定是垃圾，可以回收）

# 内存泄漏问题
c++出现内存问题大致有几个方面：
1. 缓冲区溢出
2. 空悬/野指针
3. 重复delete
4. 内存泄漏
5. 不配对的new[]/delete
6. 内存碎片（唯独这个智能指针无法解决）

# 条件变量
条件变量是非常底层的同步原语，很少直接使用，一般都是用它来实现高层的同步措施（CountDownLatch,BlockingQueue）

- #define MutexLockGuard(x) static_assert(false,"missing mutex guard var name")
- 这句话的作用主要是为了防止创建临时变量如MutexLockGuard(mutex)，正确写法为：MutexLockGuard lock(mutex);

# sleep(3)不是同步原语
sleep最好只能出现在测试代码当中，不推荐实际代码使用，可以使用timer，即IO事件回调或者条件变量。

# socket RAII
可以将socket包装成一个类，读写操作均通过此对象进行，这样一来，socket就不会出现被误关闭的现象。
- 引申：不推荐将fd1和fd2，即标准输出和标准错误关闭，防止他们被网络连接占用，造成对方收到莫名其妙的数据。正确的办法是将其定向到磁盘文件，也不推荐/dev/null，防止丢失重要信息。