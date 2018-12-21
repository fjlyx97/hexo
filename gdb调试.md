---
title: gdb调试
date: 2018-06-13 21:21:46
tags:
- 编程
- 学习
- C家族
- 调试
categories : "C家族"
---

> - 在linux下面使用gdb进行断点调试
> - 介绍如何操作gdb

<!--more-->

# 编译文件
- 平时调试代码直接使用gcc或g++即可编译
```bash
gcc test.c -o test
```
- 需要使用gdb编译需要加上-g参数
```bash
gcc -g test.c -o test
```

# 调试文件
- 使用gdb命令打开文件
```bash
gdb test
```

## 常用命令

### 设置断点
- 使用b设置断点
```
b main //在main函数设置断点
```

### 运行命令
- 使用r运行程序

### 单步运行
- 使用n进入下一行（不进入的单步执行）
- 使用s为进入的单步执行，finish为单步跳出

### 显示变量
- p可以显示变量名
```
p 变量名
```

### 查看代码
- 使用list，简写l

### 查看断点，监视
- info b //断点
- info watch //监视

### 运行到下一个断点
- continue 简写为c

### 终止运行
- 使用kill来终止

### 删除断点
- 使用delete breakpoint 1 //需先使用info进行查看

### 检查汇编
- disas 查看汇编代码

### 查看寄存器内容
- info registers 查看寄存器的值，可以用print $name 打印寄存器内容

### 每步执行
- stepi

### 检查寄存器或某个地址
- x $rsp