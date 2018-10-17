---
title: 简易Socket编程
date: 2018-02-28 19:57:54
tags:
- 编程
- 学习
- 网络编程
categories : "socket编程"
---

> - 进行简单的socket编程尝试
> - 使用C++和python分别编写服务端与客户端
> - 简单传输文本
> - 代码编写随意，仅供回忆

<!--more-->

# 编程步骤

## 服务端
1. 加载套接字(WSAStartup())
2. 绑定套接字到IP和端口上(bind())
3. 设置为监听模式(listen())
4. 接收到请求后，返回本次请求所对应的套接字(accept())
5. 发送与接受数据(send()/recv())
6. 关闭套接字(WSACleanup())

## 客户端
1. 加载套接字(WSAStartup())
2. 发出连接请求(connect())
3. 发送与接受数据(send()/recv())
4. 关闭套接字(WSACleanup())

# 记录代码

## C++服务端
```cpp
#include <WinSock2.h>
#include <iostream>
/*静态加入Lib文件，编译时需加上-lwsock32*/
#pragma comment(lib,"ws2_32.lib")
int main(void)
{
	WORD sockVersion = MAKEWORD(2, 2);		//指定socket版本号
	WSADATA wsaData;

	/*初始化socket库*/
	if (WSAStartup(sockVersion, &wsaData)!=0)
	{
		return 0;
	}

	/*创建套接字*/
	/*第一个参数指定IPv4，第二个参数指定流式传输，适用tcp，第三个参数指定tcp协议，设置为0将自动判断*/
	SOCKET slisten = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

	sockaddr_in sin;
	sin.sin_family = AF_INET;	//指定协议
	sin.sin_port = htons(6000);	//必须采用网络数据格式，使用htons来包装
	sin.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	/*进行绑定*/
	if (bind(slisten, (SOCKADDR*)&sin, sizeof(sin)) == SOCKET_ERROR)
	{
		std::cout << "bind error ! " << std::endl;
	}

	/*5为等待连接数目*/
	if (listen(slisten, 5) == SOCKET_ERROR)
	{
		std::cout << "listen error" << std::endl;
	}
	
	/*接受数据*/
	SOCKET sClient;
	sockaddr_in remoteAddr;
	char revData[255];
	int nAddrlen = sizeof(remoteAddr);
	while (true)
	{
		std::cout << "等待连接" << std::endl;
		sClient = accept(slisten, (SOCKADDR*)&remoteAddr, &nAddrlen);
		std::cout << "连接成功" << std::endl;
		send(sClient, "连接成功" , sizeof("连接成功"), 0);
		int ret = 1;    //为下面接受数据判断
		while (ret > 0)
		{
			ret = recv(sClient, revData, 255, 0);
			std::cout << revData << std::endl;
		}
	}

	return 0;
}
```

## python服务端
```python
import socket
HOST = "127.0.0.1"
PORT = 50007
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind((HOST,PORT))
s.listen(1)
print("服务器正在运行\n")
while True:
    conn,addr=s.accept()
    print("connected by %s" %{addr})
    conn.send(b'hello')
    while True:
        data = conn.recv(1024)
        print(data.decode("utf-8"))
        input_data = input("")
        conn.send(bytes(input_data,"utf-8"))
```

## python客户端
```python
import socket
import chardet
HOST = "127.0.0.1"
PORT = 6000
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect((HOST,PORT))
while True:
    rec = s.recv(1024)
    L = chardet.detect(rec)
    code = L['encoding']
    print(rec.decode(code))
    cmd = input('Please input data:')
    cmd = bytes(cmd,"gb2312")
    s.send(cmd)
    s.close()
```

