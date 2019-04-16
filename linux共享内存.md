---
title: linux共享内存
date: 2019-04-16 23:35:50
tags:
- linux
- 学习
- c家族
categories : "C家族"
---

> - linux c++共享内存的通信
> - 涉及到共享内存的创建，销毁等等

# 共享内存操作工具
系统中存在的共享内存信息，都可以通过共享内存操作工具ipcs来查看
```cpp
ipcs -m //m选项可以查看系统中相应共享内存的基本信息，输出如下
/*
------------ 共享内存段 --------------
键         shmid      拥有者      权限        字节     连接数       状态      
0x00000000 65536      lightdm    600        524288     2          目标       
0x00000000 163841     lightdm    600        524288     2          目标       
0x00000000 262146     lightdm    600        524288     2          目标       
0x00000000 294915     lightdm    600        1048576    2          目标       
0x00000000 327684     lightdm    600        67108864   2          目标       
*/

ipcrm -m (shmid) //shmid为ipcs -m命令得到的，将系统中创建的共享内存删除
```

# 共享内存的创建
## shmget接口（创建）
系统提供shmget接口来创建一片共享内存，定义如下：
```cpp
#include <sys/ipc.h>
#include <sys/shm.h>
int shmget(key_t key,int size,int shmflg);
```
- key参数可以为共享数据段命名
- size参数为申请的内存大小（字节为单位）
- shmflg为权限表标志，如0666，则创建一片权限为066的共享内存。同样还有两个参数IPC_CREAT,IPC_EXCL，其中IPC_CREAT代表如果共享内存**不存在**，则创建。IPC_EXCL则代表如果内存已经存在，返回值为-1，三者可以通过|符号相连，如IPC_CREAT|IPC_EXCL|0666。
- 返回值为共享内存的id，调用失败返回-1

## shmat接口（映射）
系统提供shmat接口来将程序与共享内存相连接，定义如下：
```cpp
void *shmat(int shm_id, const void *shm_addr, int shmflg);
```
- shm_id参数为之前shmget函数调用得到的返回值
- shm_addr参数为共享内存连接到当前进程的位置，一般写NULL，让系统指定
- shmflg参数为标志位，有三种选择。SHM_RDONLY代表共享内存只读，另外两种不常用，不过此标志位通常写0
- 返回值为共享内存首地址，调用失败返回-1

## shmdt接口（分离）
系统提供shmdt接口来将程序与共享内存相分离（有连接有分离），定义如下：
```cpp
int shmdt(const void *shmaddr);
```
- shmaddr为共享内存首地址，通常为shmat的返回值
- 调用成功返回0，失败返回-1

## shmctl接口（控制）
系统提供shmtl接口来将控制共享内存，接口定义如下：
```cpp
#include <sys/ipc.h>
#include <sys/sem.h>
int shmctl(int shmid,int cmd,struct shmid_ds *buf);
```
- shmid通常为shmget函数返回的共享内存标识符
- cmd为要采取的操作，通常有五种，IPC_STAT为根据传入的共享内存id，将获取到的信息存储到buf。IPC_SET为设置共享内存信息，通过buf传入。IPC_RMID则为删除共享内存，buf可以传入0。最后两个是SHM_LOCK,SHM_UNLOCK，为锁住共享内存块，通常为ROOT才能操作，不允许其他用户使用
- buf则通过cmd来随机应变
- 调用成功返回0，失败返回-1

# 共享内存实例
共享内存维护一个结构体，可以存储字符数组，一个进程负责往共享内存中写入数据，一个进程负责往共享内存中读出数据。

## read
```cpp
#include <iostream>
#include <cstring>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

using namespace std;

struct shared_use_st
{
    int written; // 0 write or read
    char text[2048];
};

int main()
{
    bool isRunning = 1;
    void * shm = NULL;
    struct shared_use_st* shared;
    int shmid = shmget((key_t)1234,sizeof(struct shared_use_st),0666|IPC_CREAT);
    if (shmid == -1)
    {
        cout << "Error...." << endl;
        return -1;
    }

    shm = shmat(shmid,0,0);
    if (shm == (void*)-1)
    {
        cout << "shm error....." << endl;
        return -1;
    }

    shared = (struct shared_use_st*)shm;
    shared->written = 0;
    while (isRunning)
    {
        if (shared->written != 0)
        {
            cout << "You wrote : " << shared->text << endl;
            sleep(1);
            shared->written = 0;
            if (strncmp(shared->text,"end",3) == 0)
                isRunning = false;
            else
                sleep(1);
        }
    }

    if(shmdt(shm) == -1)
    {
        cout << "Shmdt error...." << endl;
        return -1;
    }
    if (shmctl(shmid,IPC_RMID,0) == -1)
    {
        cout << "Shmctl error...." << endl;
        return -1;
    }

    return 0;
}
```

## write
```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

using namespace std;

struct shared_use_st
{
    int written; // 0 write or read
    char text[2048];
};

int main()
{
    bool isRunning = 1;
    void * shm = NULL;
    struct shared_use_st* shared;
    int shmid = shmget((key_t)1234,sizeof(struct shared_use_st),0666|IPC_CREAT);
    char buffer[1024];
    if (shmid == -1)
    {
        cout << "Error...." << endl;
        return -1;
    }

    shm = shmat(shmid,0,0);
    if (shm == (void*)-1)
    {
        cout << "shm error....." << endl;
        return -1;
    }

    shared = (struct shared_use_st*)shm;
    while (isRunning)
    {
        while (shared->written == 1)
        {
            sleep(1);
            cout << "waiting...." << endl;
        }
        cout << "Enter some text:";
        fgets(buffer,sizeof(buffer),stdin);
        strcpy(shared->text,buffer);
        cout << "Write Success" << endl;
        shared->written = 1;
        if (strncmp(buffer,"end",3) == 0)
            isRunning = false;
    }

    if(shmdt(shm) == -1)
    {
        cout << "Shmdt error...." << endl;
        return -1;
    }

    return 0;
}
```

# 安全性
共享内存的没有提供同步机制，但是我们可以通过特殊手段，如互斥量等等进行保持同步，后续会补上使用互斥量+共享内存实现进程通信
