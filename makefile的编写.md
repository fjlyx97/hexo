---
title: makefile的编写
date: 2018-06-13 21:51:29
tags:
- 学习
- 编程
- C家族
- makefile 
categories : "C家族"
---

>- 在linux下一旦需要编译的文件过多
>- 使用gcc过于繁琐，可以通过编写makefile文件来减轻工作量
>- 记录如何编写makefile

<!--more-->

# 创建makefile文件
- 首先在工程目录当中创建一个makefile文件
- 早期makefile首字母必须大写，不过现在大小写均可

# 基本makefile格式
- 编写格式如下：
```
target: dependencies
        command
//dependencies为需要的依赖
//command为需要的命令
```

- 示例如下
```makefile
test: HelloWorld.cpp
    g++ HelloWorld.cpp -o test
```

# make编译
- 使用make命令进行直接编译

# 多个文件编写makefile
- 假设当前目录下拥有三个文件
```
main.c
tool.c
tool.h
```
- 编写如下makefile
```
main: main.c tool.o
    gcc main.c tool.o -o main
//但是tool.o并不存在所以需要告知编译器tool.o位置
tool.o: tool.c
    gcc -c tool.c
//-c代表直接编译成.o的文件
```

## 常见后端文件用途
```
Windows下：
h：头文件，给编译器用来检查语法
lib：主要包含了如何找到函数的地址的信息，以及附带一些编译了一半的二进制数据
obj：编译了一半的二进制数据
dll、exe：可以运行

linux下：
.o,是目标文件,相当于windows中的.obj文件 
.so 为共享库,是shared object,用于动态连接的,相当于windows下的dll 
.a为静态库,是好多个.o合在一起,用于静态连接 
```
### lib和DLL的区别
- lib是编译时需要的，dll是运行时需要的。缺少dll文件程序将无法运行（动态链接库），而lib文件则是静态链接库

### obj文件
- obj里存的是编译后的代码跟数据，并且有名称，所以在连接时有时会出现未解决的外部符号的问题。当连成exe后便不存在名称的概念了，只有地址。lib就是一堆obj的组合。

## 删除指令
- 使用clean指令进行删除不必要的文件
```
clean:
    rm *.o main
```
- 运行make clean进行删除文件

# 在makefile当中设置变量
- 设置变量的方式与普通编程语言方式差别不大，如下例子
```
CC = gcc
```
- 具体使用的时候需要%符号
```
main: main.c test.o
    $(CC) main.c test.o -o main
```

# 编写两个可执行文件
- 默认情况下似乎无法同时编译出两个可执行文件
- 假设需要编写两个可执行文件，文件名为main_max,main_min
- 须在开头加入：
```
all: main_max main_min
```