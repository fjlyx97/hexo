---
title: linux中c++架构实战（二）
date: 2019-01-22 20:31:17
tags:
- linux
- 学习
- c家族
categories : "Linux"
---

> - linux c++架构实战系列课程笔记（二）
> - 服务器框架初步

<!-- more-->

# 服务器框架初步
## 目录结构规划
```
- nginx：主目录
- _include目录： 专门存放各种头文件
- app目录： 放主应用程序.c，如main函数以及比较核心的目录.
    link_obj：临时目录，存放临时的.o文件，不手工创建，后续使用makefile脚本创建
    dep:临时目录，会存放临时的.d开头的依赖文件，依赖文件能告知系统那些相关的文件发生变化，需要重新编译
    nginx.c：主文件，main()入口函数
    ngx_conf.c，普通源码文件
- misc目录：专门存放杂合性不好归类的C文件
- net目录：专门存放和网络处理相关的文件
- proc目录：专门存放和进程处理有关的多个文件
- signal目录：专门用于存放有关信号处理的文件
    ngx_signal.c
```

## Makefile编写
- makefile:编译项目的入口脚本，起到总体控制
- config.mk：配置脚本，应付一些可变的东西
- common.mk 最核心的编译脚本，定义makefile的编译规则，依赖规则等，通用性很强，各个子目录中都用到这个脚本

## 基础配置读取
使用配置文件，使我们的服务器程序有了极大的灵活性，使我们作为服务器程序开发者必须首先搞定的问题

配置文件：文本文件，里边除了注释行之外不要用中文，只在配置文件中使用字母，数字，下划线。以#开头的为注释行（注释行可以有中文）

## 设置程序标题（ps命令查看）
linux下可以使用env查看环境变量，C++同样可以获取到环境变量，代码如下：
```
#include <iostream>
using namespace std;
extern char** environ;
int main()
{
    cout << environ[0] << endl; //输出USER=ubuntu
    return 0;
}
```

## 实现思路
1. 重新分配一块内存，用来保存environ内存的内容，实现思路如下：
```cpp
#include <iostream>
#include <cstring>
using namespace std;
extern char** environ;
void initSetProcTitle()
{
    cout << "Old :: " << endl;
    for (int i = 0 ; environ[i] ; i++)
    {
        cout << environ[i] << endl;
    }
    int i;
    int g_environlen = 0;
    for (i = 0 ; environ[i]; i++)
    {
        g_environlen += strlen(environ[i]) + 1; //末尾/0
    }

    char* gp_envmem = new char[g_environlen]; //重新new一块等价大小的内存
    char* gp_envmem_point = gp_envmem;

    for (i = 0 ; environ[i] ; i++)
    {
        size_t size = strlen(environ[i])+1; //不能拉下\0
        strcpy(gp_envmem_point,environ[i]);
        environ[i] = gp_envmem_point;
        gp_envmem_point += size;
    }
    cout << "New ::  -----------------------------------------------------" << endl;
    for (int i = 0 ; environ[i] ; i++) //重新指向了新的那片空间
    {
        cout << environ[i] << endl;
    }
    delete[] gp_envmem;
    return;
}
int main()
{
    initSetProcTitle();
    return 0;
}
```

2. 真正设置标题
首先统计所有可用空间，即argv的所有空间和environ的所有空间相加，这就是我们标题可以设置的最大长度。
```cpp
size_t temp_len = 0;
for (int i = 0 ; argv[i] ; i++)
{
    temp_len += strlen(argv[i])+1; //统计所有argv的内存总和
}
temp_len += g_environlen; //加上environ的内存总和
argv[1] = NULL; //设置数组第二个元素为null;
strcpy(argv[0],"title"); //复制标题
//将标题之后的所有数组元素清零
...
//结束设置标题
```

# 内存泄漏的检查工具
Valgrind:帮助程序员寻找程序里的BUG和改进程序性能的工具集。擅长发现内存泄露的管理工具。能发现如下问题：
1. 使用未初始化的内存
2. 使用已经释放了的内存
3. 使用超过malloc()分配的内存
4. 堆栈的非法访问
5. 申请的内存是否有释放
6. malloc/free,new/delete申请和释放内存的匹配
7. memcpy()内存拷贝函数中源指针和目标指针的重叠

## 示范
```
g++ -g -o test test.cpp
valgrind --tool=memcheck --leak-check=full --how-reachable=yes ./test
```
如果n个allocs,n-1frees就代表内存没泄露，差值为1

# 信号功能进阶
商业软件中，不用signal(),而要用sigaction()

## sigsuspend(信号集）
一旦调用了sigsuspend函数，程序阻塞在这里，等待一个信号，此时进程挂起，不占用CPU时间。只有收到信号才会被唤醒

# write函数
多个进程同时写 一个日志文件，我们看到输出结果并不混乱，是有序的；我们的日志代码应对多进程往日志文件中写时没有问题；

## 掉电导致write的数据丢失破解法
1. 直接I/O：直接访问物理磁盘。使用O_DIRECT：绕过内核缓冲区。
2. open文件时用O_SYNC选项：同步选项,把数据直接同步到磁盘.只针对write函数有效，使每次write()操作等待物理I/O操作的完成。将写入内核缓冲区的数据立即写入磁盘，将掉电等问题造成的损失减到最小；
3. 缓存同步：尽量保证缓存数据和写道磁盘上的数据一致；
- sync(void)：将所有修改过的块缓冲区排入写队列；然后返回，并不等待实际写磁盘操作结束，数据是否写入磁盘并没有保证；
- fsync(int fd)：将fd对应的文件的块缓冲区立即写入磁盘，并等待实际写磁盘操作结束返回（常用）
- fdatasync(int fd)：类似于fsync，但只影响文件的数据部分。而fsync不一样，fsync除数据外，还会同步更新文件属性；

# 网络通讯
客户端:client，服务端：server，所以C/S框架就对应：客户端/服务器。

## 角色规律总结
数据通讯在两端进行，为客户端和服务端。总有一方先发起数据包，则发起数据包的这一端成为客户端（浏览器）；被动收到数据包的这端为服务端（淘宝服务器）

一旦连接建立起来，数据双向流动，这个过程则被成为**双工**，

## OSI七层网络模型（Open System Interconnect）
物（物理层）链（数据链路层）网（网络层）传（传输层）会（会话层）表（表示层）应（应用层）。把一个要发出去的数据包从里往外裹了七层，最后将其发出去。

## TCP/IP协议四层模型
```
OSI七层模型         TCP/IP四层模型
7：应用层
6：表示层               应用层
5：会话层

4：传输层               传输层

3：网络层               网络层

2：数据链路层
1：物理层               链路层
```

## 套接字socket概念
socket就是一个数字，通过调用socket()函数生成，且这个数字具有唯一性，直到调用close()函数来把这个数字关闭。

文件描述符：Linux下一切皆可堪称文件，因此我们可以把socket也看成是文件描述符，我们可以用它来收发数据。实例代码以往文章写过，[实例代码](http://codeforme.cn/2018/02/28/%E7%AE%80%E6%98%93Socket%E7%BC%96%E7%A8%8B/#more)

02.01更新简单实例代码：
```cpp
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <cstring>
#include <cstdlib>
#include <unistd.h>

using namespace std;
int main(int argc , char** argv)
{
    int listenfd = socket(AF_INET,SOCK_STREAM,0);
    cout << "listenfd is :" << listenfd << endl;
    struct sockaddr_in serv_addr;
    memset(&serv_addr,0,sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(1234);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);

    int result;
    result = bind(listenfd,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    if (result == -1)
    {
        return -1;
    }
    result = listen(listenfd,32);
    if (result == -1)
    {
        return -1;
    }
    int confd;
    while(true)
    {
        confd = accept(listenfd,(struct sockaddr*)NULL,NULL);
        cout << "confd is : " << confd << endl;
        write(confd,"Hello World\n",13);
        close(confd);
    }
    return 0;
}
```


## TCP和UDP的区别
TCP（Transfer Control Protocol）：传输控制协议。可靠的面向连接的协议，一旦数据包丢失，操作系统层会感知并重新帮助你发送数据包。因此TCP更适用于文字传输，收发邮件准确率高，但效率相对差，适用范围场合更广
UDP（User Datagram Protocol）：用户数据报协议，不可靠无连接协议。

## 最大传输单元MTU（Maximum Transfer Unit）
MTU:每个数据包包含的数据最多可以有多少个字节，大概是1.5K左右

## TCP三次握手建立连接的过程
1. 客户端给服务器发送了一个SYN标志位置位的无包体TCP数据包，SYN被置位，就表示发起TCP连接，协议规定
2. 服务器收到了SYN标志位的数据包，因此返回给客户端一个SYN和ACK标志位都被置位的无包体TCP数据包
3. 客户端收到数据包之后，再次发送ACK置位的数据包，服务器端收到之后，客户端服务器的TCP连接正式建立

## telnet工具使用介绍
命令行方式运行客户端TCP通讯工具，可以连接到服务器端，往服务器端发送数据，也可以接收服务器端发过来的信息。这个工具可以方便的测试服务器端的某个TCP端口是否连通，可以正常的收发数据。
```
telnet 192.168.1.126 9000
```

## TCP断开的四次握手
1. FIN,ACK 服务器->客户端
2. ACK 客户端->服务器
3. FIN,ACK 客户端->服务器
4. ACK 服务器->客户端

## TCP状态转换
同一个IP，同一个端口，只能被成功bind()一次，若再次bind()就会失败，并且显示:Address already in use，使用netstat命令显示网络相关信息
```
netstat -anp //a 显示所有选项 n能显示成数字全部显示成数字 p 显示段落对应的程序名
```
- 只要客户端连接到服务器，并且服务器把客户端关闭，服务器端就会产生一条针对9000监听端口，状态为TIME_WATI的连接。那么此时重启服务器，就会失败，bind()函数报错:Address already in use

一个TCP连接可能处于11种状态
- 客户端：CLOSED ->SYN_SENT->ESTABLISHED【连接建立，可以进行数据收发】
- 服务端：CLOSED ->LISTEN->【客户端来握手】SYN_RCVD->ESTABLISHED【连接建立，可以进行数据收发】
- 服务器主动关闭连接：ESTABLISHED->FIN_WAIT1->FIN_WAIT2->TIME_WAIT
- 客户端被动关闭：ESTABLISHED->CLOSE_WAIT->LAST_ACK
任一方close连接，就会发送FIN标志置位的数据包

具有TIME_WAIT状态的TCP连接，就好像是一种残留信息一样，拥有时间限制，大概为1-4分钟（2MSL 最长数据包生命周期）
### 引入TIME_WAIT状态的原因
1. 可靠的实现TCP全双工的终止
2. 允许老的重复的TCP数据包 

### RST标志
对于每个TCP连接，操作系统开辟一个接收缓冲区，一个发送缓冲区来处理数据。当我们close连接时，会先发送清空缓冲区，再发送fin包关闭连接。而RST标志则表示异常关闭，一般都会导致丢失一些数据包。
- 使用setsockopt(SO_LINGER)选项要是开启，发送的就是RST包，此时发送缓冲区的数据就会被丢弃，不会进入TIME_WAIT状态

## SO_REUSEADDR
setsockopt(SO_RESUEADDR)用在服务器端，socket()创建之后，bind()之前
1. SO_REUSEADDR允许启动一个监听服务器并捆绑其端口，即便以前建立的**连接**存在，也允许使用同样的IP和端口进行创建
2. 允许同一个端口上启动同一个服务器的多个实例，只要每个实例捆绑一个不同的本地IP地址即可
3. 允许单个进程捆绑同一个端口到多个套接字，只要每次捆绑指定不同的本地IP地址即可
4. 允许完全重复绑定：当一个IP地址和端口已经绑定到某个套接字上时，如果传输协议支持，同样的IP地址和端口还可以绑定到另一个套接字上，一般仅支持UDP套接字。
- 所有TCP服务器都应该指定本套接字选项，以防止套接字处于TIME_WAIT时bind()失败的情况出现

# listen()队列剖析
listen()：监听端口，用在TCP连接中服务端角色
- 调用格式int listen(int sockfd , int backlog);backlog的数字为队列维护的最大条目数

## 监听套接字队列
对于一个调用listen()进行监听的套接字，操作系统会给这个套接字维护两个队列
1. 未完成连接队列，即半连接所处位置
2. 已完成连接队列，即已完成三次握手，已变成ESTABLISHED状态的连接

## RTT超时时间
RTT是未完成队列中任意一项在未完成队列中留存的时间，这个时间取决于客户端和服务器；
对于客户端，这个RTT时间是第一次和第二次握手加起来的时间；
对于服务器，这个RTT时间实际上是第二次和第三次握手加起来的时间；
- 如果有人故意不发送三次握手中第三个ACK包，那么连接将在未完成队列中存在大约75秒，如果超过时间，就会被操作系统杀掉

## accept函数
accept()函数，就是从**已完成**连接队列中的队首，取出一项，返回给进程。如果队列为空，则会阻塞，一直到已完成队列中有一项才会被唤醒
- accept()返回的是个套接字，这个套接字就代表那个已经用三次握手建立起来的那个tcp连接，因为accept()是从已完成队列中取的数据；

## 思考
1. 如果两个队列之和【已完成连接队列，和未完成连接队列】达到了listen()所指定的第二参数，也就是说队列满了。此时，再有一个客户发送syn请求，服务器怎么反应？
- 答：服务器会忽略这个syn包，不给回应；客户端发现syn包没回应，会重发syn包
2. 如果连接处于未被accept取走，呆在已完成连接队列中，但是客户端发送来数据，那么数据存于何处？
- 答：数据为被保存在已经连接的套接字的接收缓冲区里，这个缓冲区的大小决定最大能接收到多少数据量

## syn攻击（syn flood）
典型利用TCP/IP协议涉及弱点进行攻击的行为（拒绝服务攻击）
- backlog：进一步明确和规定了：指定给定套接字上内核为之排队的最大已完成连接数【已完成连接队列中最大条目数】；

# 阻塞与非阻塞I/O （对比同步与异步I/O）
阻塞和非阻塞主要是调用系统函数时，这个函数会导致我们进程进入休眠状态，整个流程不继续往下执行。而同步与异步则有点类似。通过查阅资料可以看出差别：
> - 老张把水壶放到火上，立等水开。（同步阻塞）
> - 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）
> - 老张把响水壶放到火上，立等水开。（异步阻塞）
> - 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）

# epoll技术简介
## epoll概述
1. I/O多路复用，epoll的最大特点就是支持高并发。传统多路复用技术select,poll，在并发量达到1000-2000，性能就会明显下降。epoll则支持上百万并发量，因此epoll在Linux内核2.6引入。
2. epoll技术不会因为并发量增多而导致效率明显下降，但是会消耗一定的内存去保存这个连接相关的数据。（并发量总还是有限制的，并不可能是无限的）
3. 假设10万连接，可能只有上百个客户端给你发送数据，那么epoll只处理这上百个客户端，而select,poll则会全部判断。
4. epoll事件驱动机制，在单独进程或者单独线程中运行，收集或处理事件，没有进程或线程之间切换的消耗，十分高效。

## 相关函数
### epoll_create()函数
```cpp
int epoll_create(int size);
```
创建一个epoll对象，返回该对象的描述符（文件描述符），这个描述符就代表epoll。这个对象最终要调用close()，关闭文件句柄。
- size目前大于0即可
- struct eventpoll *ep = (struct eventpoll*)calloc(1, sizeof(struct eventpoll)); 
- rbr结构成员：代表一颗红黑树的根节点（刚开始指向空）,把rbr理解成红黑树的根节点的指针；
- 总结：创建了一个eventpoll结构对象，被系统保存起来；

### epoll_ctl()函数
```cpp
int epoll_ctl(int efpd,int op,int sockid,struct epoll_event* event);
```
把一个socket以及这个socket相关的事件，添加到这个epoll对象描述符中去，目的就是通过这个epoll对象来监视这个socket（客户端的TCP连接数据的来往情况）;
- efpd:epoll_create()返回的epoll对象描述符
- op:动作，添加/删除/修改，对应数字1，2，3，如EPOLL_CTL_ADD,EPOLL_CTL_DEL,EPOLL_CTL_MOD。
```
//EPOLL_CTL_ADD添加事件：等于你往红黑树上添加一个节点，每个客户端连入服务器后，服务器都会产生一个对应的socket，每个连接这个socket值都不重复。所以，这个socket就是红黑树中的key，把这个节点添加到红黑树上去；
//EPOLL_CTL_MOD：修改事件.只有用了EPOLL_CTL_ADD把节点添加到红黑树上之后，才存在修改；
//EPOLL_CTL_DEL：从红黑树上把这个节点删掉,这会导致这个socket【这个tcp链接】上无法收到任何系统通知事件；
```
- sockid:表示客户端连接，accept()返回值，这个就是红黑树里面的key
- event:事件信息,如上EPOLL_CTL_ADD等事件信息。

原理：
- epi = (struct epitem*)calloc(1, sizeof(struct epitem));
- epi = RB_INSERT(_epoll_rb_socket, &ep->rbr, epi); 【EPOLL_CTL_ADD】增加节点到红黑树中
- epitem.rbn ，代表三个指针，分别指向红黑树的左子树，右子树，父亲；
- epi = RB_REMOVE(_epoll_rb_socket, &ep->rbr, epi);【EPOLL_CTL_DEL】，从红黑树中把节点干掉
- EPOLL_CTL_MOD，找到红黑树节点，修改这个节点中的内容；

红黑树的节点是epoll_ctl(EPOLL_CTL_ADD)往里增加的节点，红黑树的节点是epoll_ctl(EPOLL_CTL_DEL)删除的，面试可能考

### epoll_wait()函数
```cpp
int epoll_wait(int epfd,struct epoll_event* events,int maxevents,int timeout);
```
阻塞一小段时间并等待事件发生，返回事件集合，也就是获取内核的事件通知
- efpd:epoll_create()返回的epoll对象描述符
- events:是内存，也是数组，长度是maxevents，表示此次epoll_wait调用可以收集到的事件
- timeout：阻塞事件

## 内核向双向链表中增加节点
1. 客户端完成三次握手，服务器需要accept()
2. 客户端关闭连接，服务器也要调用close()关闭
3. 客户端发送数据来的适合，服务器需要调用read(),recv()函数来收数据
4. 当可以发送数据时，服务器调用send(),write()

# 多线程
- POSIX: 表示可移植操作系统接口（Portable Operating System Interface of UNIX）
- POSIX线程：是POSIX的线程标准，定义了创建和操作线程的一套API，一般以pthread_开头，比较成熟方便。

## 线程池
提前创建好一堆线程，并使用一个类来统一管理调度这一堆线程，这一堆线程我们就叫做线程池
1. 事先创建好线程，避免动态创建线程来执行任务，提高了程序的稳定性
2. 提高程序运行效率：线程池中线程，反复循环再利用