---
title: C++网络编程
date: 2020-01-18 17:45:03
tags:
- 编程
- 学习
- C家族
categories : "C家族"
---

> - 根据C++百万并发网络通信引擎架构与实现（Socket、全栈、跨平台）课程所记录
> - 记录网络编程细节

<!--more-->

# 发送结构化的网络消息数据
假设有如下结构体：
```cpp
struct DataPack
{
    int age;
    char name[32];
};

//使用如下方法发送
DataPack db = {80,"Yi Yi"};
send(sClient, (const char*)&db, sizeof(DataPack) , 0);  

//使用如下方法接收
char recData[255];
int ret = recv(sclient, recData, 255, 0);
if (ret > 0)
{
    DataPack* dp = (DataPack*)recData;
    std::cout << dp->age << std::endl;
    std::cout << dp->name << std::endl;
}
```
- 必须保证字节序是一样的，如long大小不同平台大小不一样，需要特别注意
- 不严谨，不推荐这么使用

# 网络数据报文格式定义
- 包头：描述消息包大小，描述数据作用
- 包体：数据
```cpp

enum CMD
{
    CMD_LOGIN,
    CMD_LOGINOUT，
};

//消息头
struct DataHeader
{
    short dataLength; //数据长度
    short cmd;        //数据命令
};

//消息体
struct Login
{
    char userName[32];
    char passWord[32];
};

struct LoginResult
{
    int result;
};

//接收部分代码
DataHeader header;
int nLen = recv(_cSock,(const char*)&header,sizeof(DataHeader),0);
//根据接收部分另外编写代码
switch (header.cmd)
{
    case CMD_LOGIN:
    {
        Login login = {};
        recv(_cSock,(const char*)&login,sizeof(Login),0);
    }
    ...
}

```

## 多次收发修改为一次收发
```cpp
struct DataHeader
{
    short dataLength;
    short cmd;
}
struct Login:public DataHeader
{
    Login()
    {
        this->dataLength = sizeof(Login);
        this->dataLength = CMD_LOGIN;
    }
    char userName[32];
    char passWord[32];
}
```

# 粘包
连续发送大量数据时，如果没有及时接收，会造成缓冲区慢。造成缓冲区溢出，无法发送，网络阻塞。
## 解决办法
程序当中设置自己的缓冲区，及时将数据从系统底层缓冲区中提取出来。

# Select网络模型
```cpp
//对于windows中来说，nfds无所谓可以写0
WINSOCK_API_LINKAGE int WSAAPI select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,const PTIMEVAL timeout);
```