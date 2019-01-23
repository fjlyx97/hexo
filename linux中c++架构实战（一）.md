---
title: linux中c++架构实战（一）
date: 2018-12-28 19:33:17
tags:
- linux
- 学习
- c家族
categories : "Linux"
---

> - linux c++架构实战系列课程笔记（一）
> - 涉及nginx开发初步
> - 终端进程的基本概念
> - 信号的基本概念
> - fork函数，僵尸进程，守护进程

<!-- more-->

# nginx目录结构
```
nginx-1.14.2
├── auto            //一些自动化脚本
├── CHANGES         //版本更新说明
├── CHANGES.ru      //俄语版本更新说明
├── conf            //默认配置文件目录
├── configure       //编译nginx之前必须先执行的脚本文件
├── contrib         //脚本和工具，如VIM高亮
├── html            //存放两个起始界面的html文件
├── LICENSE         //许可证
├── man             //包含man手册，可以直接使用man调用
├── README          //说明文件
└── src             //源文件
```

## auto目录
```
auto
├── cc              //检查编译器脚本
├── lib             //检查依赖库脚本
├── os              //检查操作系统类型的脚本
├── types           //检查平台类型的脚本
```

## src目录
```
src
├── core            //核心代码
├── event           //事件模块相关代码
├── http            //http(web)服务相关代码
├── mail            //邮件相关代码
├── misc            //辅助代码，测试C++头文件兼容性
├── os              //操作系统相关代码
└── stream          //流处理相关代码
```

# 编译nginx
## 一、安装依赖
- nginx内核版本2.6以上
- gcc编译器
- pcre函数库，用以支持正则表达式
- zlib库，解压缩功能
- openssl库，ssl相关功能库
```
apt install build-essential libpcre3-dev libz-dev libssl-dev
```
## 二、用configure进行配置安装
- --prefix:指定最终安装到的目录，默认为：/usr/local/nginx（父目录）
- --sbin-path 指定可执行文件目录，为prefix的子目录，默认为：sbin/nginx
- --conf-path 指定配置文件目录，为prefix的子目录，默认为：conf/nginx.conf
- --with为缺省不编译，--without为缺省编译
```
./configure //直接缺省配置
make
make install
```

## 三、nginx的启动和简单使用
```
cd /usr/local/nginx/sbin
./nginx
```

# nginx整体结构
## ps -ef简要说明
- 第一列：UID 进程所属用户ID
- 第二列：进程ID（PID），用来唯一的标识一个进程
- 第三列：父进程ID（GPID），被fork创建出来的子进程

## nginx进程模型
一个master有一到多个worker进程，这种工作机制是对外服务的。这种工作机制保证了nginx能够稳定灵活的运行

master进程责任：监控进程，本身不处理业务，监控worker进程

worker进程责任：用来干主要的活，处理用户业务

master进程和用户进程通讯，可以用信号，或者是共享内存

稳定性：workder进程一旦挂掉，master进程会立刻fork一个新进程投入到工作中去

## 调整worker进程数量
公认的做法：多核计算机，让每个worker运行在一个单独的核上，最大限度减少CPU切换成本，提高稳定性
```
lscpu //查看CPU数据
```

- 在nginx目录下的conf文件夹中，修改nginx.conf配置文件，修改worker_process 

# 终端和进程的关系
## 终端与bash进程
使用如下命令查看bash
```
ps -ef | grep bash
```
可以观察到拥有pts/0，pts代表虚拟终端，/后面为编号

- 每个进程还属于一个进程组，一个或者多个进程的集合，每个进程组拥有一个唯一的进程ID，可以使用系统函数来调用或者加入进程组（PGRP）
- 会话（session）是一个或者多个进程组的集合，而这个会话有一个session leader，那么这个bash（shell）通常就是session leader。但是可以调用系统函数调用新session

## strace工具的使用
```
sudo strace -e trace=signal -p 进程id
```

# 忽略SIGHUP信号
## 使用signal.h进行忽略信号
引入头文件signal.h，使用对应的signal函数进行忽略信号
```cpp
#include <iostream>
#include <unistd.h>
#include <signal.h>
using namespace std;
int main()
{
    signal(SIGHUP,SIG_IGN); //系统函数，告诉操作系统忽略SIGHUP信号，不要杀死进程
    for (;;)
    {
        sleep(1);
        cout << "Sleep 1 sec" << endl;
    }
    return 0;
}
```
- 一旦程序允许，退出终端，程序tty变为？且ppid变为1（祖宗节点），被称为孤儿进程

## setsid
### setsid函数
setsid不适合进程组组长调用，程序退出时，bash会往相同的Session里面发送SIGHUP信号，因此可以使用fork函数进行创建进程
```cpp
#include <iostream>
#include <unistd.h>
using namespace std;
int main()
{
    pid_t pid;
    pid = fork(); //出现子进程
    if (pid < 0){
        cout << "Fork Error" << endl;
    }
    else if (pid == 0){
        cout << "Son" << endl;
        setsid(); //新建独立的session，但进程组组长调用setsid时无效的
        for (;;)
        {
            sleep(1);
            cout << "Sleep 1 sec" << endl;
        }
        return 0;
    }
    else{
        cout << "Father" << endl; //pid>0
        return 0;
    }
    return 0;
}
```
### setsid命令
启动一个进程，而且能够使其在新的session中，终端退出不退出
```
setsid ./可执行程序
```

### nohup命令
```
nohup ./可执行程序 &
```

# 信号的基本概念
进程之间的常用通信手段：发送信号,kill
- 信号：通知（事件通知）用来通知某个进程发生了某个事情
- 名字一般以SIG开头
- 信号既有名字，也是一些正整数常量（宏定义），需要包含signal.h头文件，在/usr/include

## 信号的产生
1. 由某个进程发送给另外一个进程或发送给自己
2. 由内核（操作系统）发送给某个进程

## kill命令
kill工作是发个信号给进程，可以发送多种信号
- kill -数字 进程id //能发出这个数字对应的信号，可以通过查看头文件获取定义

# Unix/Linux操作系统体系结构
类Unix操作系统体系结构分为两个状态 （1）用户态 （2）内核态

# signal函数
信号来了之后，我们可以选择进行捕捉或者忽略，signal第一个参数为要捕捉的信号，第二个参数指定函数名，代表捕捉到信号后要执行的函数

## 可重入函数
在信号处理函数当中，调用它的安全的。这些函数可重入，被称为异步信号安全。
- 有一些周知的函数都是不可重入的，如malloc、printf等

### 处理函数注意事项
1. 在信号处理的函数当中，尽量做简单的事情，尽量不要调用系统函数避免引起麻烦
2. 如果一定要调用系统函数，调用可重入的系统函数，或异步信号安全的函数（有表可查）
3. 如果一定要调用可能修改errno的值的可重入系统函数，那么必须备份errno的值，并调用结束之后进行恢复
- 一旦在信号处理中使用了不可重入函数，将会导致程序错乱（绝对不能犯！！）
- signal因为兼容性，历史原因，官方不建议使用，建议使用sigaction函数代替

# 信号集
一个进程，必须能够记住这个进程当前阻塞了哪些信号，linux中用sigset_t结构类型来表示信号集，一个进程对应一个信号集

## 信号相关函数
1. sigemtpyset()把信号集中的所有信号清零，表示这60多个信号都没有出现
2. sigfillset()把信号集中所有信号设置为1，跟sigemptyset()相反
3. 用sigaddset()，sigdelset()就可以往信号集中增加信号，或者从信号集中删除指定信号
4. sigprocmask(),sigismember()。sigprocmask()函数能够设置该进程对应的信号集中的内容，sigismember测试信号是否被屏蔽（为1）
样例代码如下：
```cpp
#include <stdio.h>
#include <stdlib.h>  //malloc
#include <unistd.h>
#include <signal.h>

//信号处理函数
void sig_quit(int signo)
{   
    printf("收到了SIGQUIT信号!\n");
    if(signal(SIGQUIT,SIG_DFL) == SIG_ERR)
    {
        printf("无法为SIGQUIT信号设置缺省处理(终止进程)!\n");
        exit(1);
    }
}

int main(int argc, char *const *argv)
{
    sigset_t newmask,oldmask; //信号集，新的信号集，原有的信号集，挂起的信号集
    if(signal(SIGQUIT,sig_quit) == SIG_ERR)  //注册信号对应的信号处理函数,"ctrl+\" 
    {        
        printf("无法捕捉SIGQUIT信号!\n");
        exit(1);   //退出程序，参数是错误代码，0表示正常退出，非0表示错误，但具体什么错误，没有特别规定，这个错误代码一般也用不到，先不管他；
    }

    sigemptyset(&newmask); //newmask信号集中所有信号都清0（表示这些信号都没有来）；
    sigaddset(&newmask,SIGQUIT); //设置newmask信号集中的SIGQUIT信号位为1，说白了，再来SIGQUIT信号时，进程就收不到，设置为1就是该信号被阻塞掉呗

    //sigprocmask()：设置该进程所对应的信号集
    if(sigprocmask(SIG_BLOCK,&newmask,&oldmask) < 0)  //第一个参数用了SIG_BLOCK表明设置 进程 新的信号屏蔽字 为 “当前信号屏蔽字 和 第二个参数指向的信号集的并集
    {                                                 //一个 ”进程“ 的当前信号屏蔽字，刚开始全部都是0的；所以相当于把当前 "进程"的信号屏蔽字设置成 newmask（屏蔽了SIGQUIT)；
                                                      //第三个参数不为空，则进程老的(调用本sigprocmask()之前的)信号集会保存到第三个参数里，用于后续，这样后续可以恢复老的信号集给线程
        printf("sigprocmask(SIG_BLOCK)失败!\n");
        exit(1);
    }
    printf("我要开始休息10秒了--------begin--，此时我无法接收SIGQUIT信号!\n");
    sleep(10);   //这个期间无法收到SIGQUIT信号的；
    printf("我已经休息了10秒了--------end----!\n");
    if(sigismember(&newmask,SIGQUIT))  //测试一个指定的信号位是否被置位(为1)，测试的是newmask
    {
        printf("SIGQUIT信号被屏蔽了!\n");
    }
    else
    {
        printf("SIGQUIT信号没有被屏蔽!!!!!!\n");
    }
    if(sigismember(&newmask,SIGHUP))  //测试另外一个指定的信号位是否被置位,测试的是newmask
    {
        printf("SIGHUP信号被屏蔽了!\n");
    }
    else
    {
        printf("SIGHUP信号没有被屏蔽!!!!!!\n");
    }

    //现在我要取消对SIGQUIT信号的屏蔽(阻塞)--把信号集还原回去
    if(sigprocmask(SIG_SETMASK,&oldmask,NULL) < 0) //第一个参数用了SIGSETMASK表明设置 进程  新的信号屏蔽字为 第二个参数 指向的信号集，第三个参数没用
    {
        printf("sigprocmask(SIG_SETMASK)失败!\n");
        exit(1);
    }

    printf("sigprocmask(SIG_SETMASK)成功!\n");
    
    if(sigismember(&oldmask,SIGQUIT))  //测试一个指定的信号位是否被置位,这里测试的当然是oldmask
    {
        printf("SIGQUIT信号被屏蔽了!\n");
    }
    else
    {
        printf("SIGQUIT信号没有被屏蔽，您可以发送SIGQUIT信号了，我要sleep(10)秒钟!!!!!!\n");
        int mysl = sleep(10);
        if(mysl > 0)
        {
            printf("sleep还没睡够，剩余%d秒\n",mysl);
        }
    }
    printf("再见了!\n");
    return 0;
}
```

# fork函数
在一个进程（程序中，可以用fork()创建一个子进程，当子进程被创建时，它从fork()指令的下一条开始执行与父进程相同的代码
```cpp
#include <iostream>
#include <unistd.h>
using namespace std;
int main()
{
    pid_t pid;
    pid = fork();
    if (pid < 0)
    {
        cout << "Fork error" << endl;
    }

    for (;;)
    {
        cout << "Thread id : " << getpid() << endl;
    }
    return 0;
}
```
- fork之后，是父进程先执行还是子进程先执行是不一定的。

# 僵尸进程（Z+）
- 僵尸进程：已经被终止，但是依旧没被内核舍弃，内核认为父进程可能需要子进程的信息。作为开发者，解决不允许僵尸进程的存在
僵尸进程的产生，在Unix系统中，一个子进程结束了，而它的父进程还存活，但该父进程没有调用（wait/waitpid）函数来进行额外的处置，那么这个子进程就会变成一个僵尸进程

## 干掉僵尸进程
- 重启电脑
- kill掉父进程
- SIGCHLD信号：当一个进程被终止的时候，这个信号会被发送给父进程
对于源码中有fork的行为，应当拦截处理SIGCHLD信号。在这里我们使用waitpid函数如下：
```cpp
//waitpid用于获取子进程的终止状态，这样子进程就不会成为僵尸进程
int status;
pid_t pid = waitpid(-1,&status,WNOHANG); //-1表示等待所有子进程，第二个参数为保存子进程信息，第三个参数提供额外选项，让这个waitpid()立即返回
if (pid == 0) //子进程未结束，会立即返回这个数字，一般不应该出现这个数字
    return;
if (pid == -1) //表示waitpid调用有错误，有错误也返回出去
    return;
return; //正常退出
```

## 进一步认识fork函数
fork产生新进程的速度非常快，且并不复制原进程的内存空间，而是和原进程一起共享内存空间，但这个内存空间可以同时被多个进程同时读写内存。但如果内存被修改的话，这个内存则会被重新复制一份
- fork会返回两次，父进程返回新建立子进程的id进程，子进程返回0

## fork失败的可能性
- 系统中进程太多（缺省最大的pid为32727）
- 每个用户拥有允许开启的进程总数

# 守护进程
一种长期运行的进程，在后台运行，且不与任何控制终端关联
## 基本特点
- 生存周期长，操作系统启动时它就启动，操作系统关闭时它就关闭（非必须，但推荐）
- 守护进程和终端无关联，不控制终端，即终端退出，守护进程不会退出

linux操作系统本身拥有很多守护进程在运行
- ppid=0；内核进程，跟随系统启动而启动，声明周期贯穿整个系统
- cmd列名字带\[\]这种，为内核守护进程
- 老祖init：系统守护进程，它负责启动各运行层次特定的系统服务；所以很多进程的ppid为init，而这个init同样收养孤儿进程
- cmd列中不带\[\]的普通守护进程（用户级守护进程）

## 守护进程的编写规则
1. 调用umask(0)，用来限制屏蔽一些文件权限。
2. fork()一个子进程，然后父进程退出。固定套路

## 守护进程不会收到的信号
1. SIGHUP信号。守护进程不会收到来自内核的SIGHUP信号，意味着如果守护进程收到了这个信号，一定是其他程序发给它的。很多通知文件把这个信号作为通知信号，表示配置文件已经发生改动，守护进程应该重新读入信号
2. SIGINT信号（终端中断符,CTRL+C）
3. SIGWINCH信号（终端窗口大小改变信号）

## 守护进程和后台进程的区别
1. 守护进程和终端不挂钩，但是后台进程可以往终端输入东西
2. 守护进程关闭终端时不受影响，而后台进程会退出

## 守护进程范例代码
```cpp
#include <stdio.h>
#include <stdlib.h>  //malloc
#include <unistd.h>
#include <signal.h>

#include <sys/stat.h>
#include <fcntl.h>

//创建守护进程
//创建成功则返回1，否则返回-1
int ngx_daemon()
{
    int  fd;

    switch (fork())  //fork()子进程
    {
    case -1:
        //创建子进程失败，这里可以写日志......
        return -1;
    case 0:
        //子进程，走到这里，直接break;
        break;
    default:
        //父进程，直接退出 
        exit(0);         
    }

    //只有子进程流程才能走到这里
    if (setsid() == -1)  //脱离终端，终端关闭，将跟此子进程无关
    {
        //记录错误日志......
        return -1;
    }
    umask(0); //设置为0，不要让它来限制文件权限，以免引起混乱

    fd = open("/dev/null", O_RDWR); //打开黑洞设备，以读写方式打开
    if (fd == -1) 
    {
        //记录错误日志......
        return -1;
    }
    if (dup2(fd, STDIN_FILENO) == -1) //先关闭STDIN_FILENO[这是规矩，已经打开的描述符，动他之前，先close]，类似于指针指向null，让/dev/null成为标准输入；
    {
        //记录错误日志......
        return -1;
    }

    if (dup2(fd, STDOUT_FILENO) == -1) //先关闭STDIN_FILENO，类似于指针指向null，让/dev/null成为标准输出；
    {
        //记录错误日志......
        return -1;
    }

    if (fd > STDERR_FILENO)  //fd应该是3，这个应该成立
    {
        if (close(fd) == -1)  //释放资源这样这个文件描述符就可以被复用；不然这个数字【文件描述符】会被一直占着；
        {
            //记录错误日志......
            return -1;
        }
    }

    return 1;
}

int main(int argc, char *const *argv)
{
    if(ngx_daemon() != 1)
    {
        //创建守护进程失败，可以做失败后的处理比如写日志等等
        return 1; 
    } 
    else
    {
        //创建守护进程成功,执行守护进程中要干的活
        for(;;)
        {        
            sleep(1); //休息1秒
            printf("休息1秒，进程id=%d!\n",getpid()); //你就算打印也没用，现在标准输出指向黑洞（/dev/null），打印不出任何结果【不显示任何结果】
        }
    }
    return 0;
}
```
1. 首先编写守护进程函数ngx_daemon，并规定返回值为1的时候，开启成功。在函数中进行fork开启分支。如果fork返回值为0代表是子进程，可以直接继续，如果返回值为-1则代表开启子进程失败，否则其他情况为父进程，则直接终止父进程。
2. 使用setsid函数来成为一个会话的leaderk，并使用umask来提高创建文件的权限
3. 使用文件指针打开空文件描述符/dev/null，并使用dup2函数将输入以及输出，错误文件描述与它相关联
4. 关闭文件指针，释放资源

# 一些概念
## 文件描述符
正数，用来标识一个文件，当打开一个文件或创建文件，操作系统都会返回一个文件描述符。后续对文件操作的函数都会用到文件描述符作为参数。linux中三个特殊的描述符，分别为0，1，2；
- 0为标准输入【键盘】，对应符号常量STDIN_FILENO
- 1为标准输出【屏幕】，对应符号常量STDOUT_FILENO
- 2为标准错误【屏幕】，对应符号常量STDERR_FILENO
一旦程序被运行起来，这三个文件描述符0，1，2被自动打开（自动指向对应设备）
## 空设备
/dev/null 一个特殊的设备文件，它丢弃一切往里面写入的数据（如黑洞一般）

# 服务器程序目录规划
## 信号高级认识范例
一旦信号处理函数当中，处理了sleep函数，那么接下来就是多次触发了同样的信号处理函数，也同样不会处理。而是等待上一个信号处理函数执行函数执行完毕才会再次执行信号处理函数。