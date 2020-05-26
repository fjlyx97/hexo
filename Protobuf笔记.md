---
title: Protobuf笔记
date: 2020-05-22 19:37:08
tags:
- Protobu
- 自动化
categories : "自动化构建"
---

> - 记录学习Protobuf笔记
> - 基于Proto3

<!--more-->

[Protobuf终极教程]("https://colobu.com/2019/10/03/protobuf-ultimate-tutorial-in-go/#%E5%8E%86%E5%8F%B2")
# 序列化
序列化的定义：将数据结构和对象的状态转化为可以存储的（文件、内存），或者传输的格式。反向序列化就是将之前的操作逆向回去。

## 常见序列化方式
1. xml格式：易于阅读，但是失去了紧凑的基于字节流编码的优势
2. json格式：广泛用于C/S端
3. yaml格式： 类似json，但是特性更强大

- Protobuf支持C++、C#、Dart、Go、Java、Python、Rust等，同时也是跨平台的。

# 简单的例子
- 第一行一般指定版本：syntax = "proto3";如果没有指定，默认为2版本
- 定义消息的格式：
```proto
syntax = "proto3";
message SearchRequest {                //SearchRequest为消息类型
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- 生成命令：protoc --go_out=. 文件名.proto
- go_out指定生成文件的保存路径，会生成:文件名.pb.go

# 序号
一般来说，前十个序号比较重要，占一个字节，超过十个字节以后就会占用两个字节。

# 引入其他proto
可以使用public,weak两个关键词进行引入，如：
```
import public "test.proto"
import weak "test1.proto"
```

# 包名
使用package关键字定义包名，防止Message字段冲突

# option
option可以用在proto的scope中，或者message、enum、service的定义中。

# repeated
这个关键字允许后面的字段重复，可以近似看成数组

# Reserved
指明Message忽略某些字段

# enum枚举类型
常用，一个字段只能取一个特定的值。注意：enum的类型是全局的，意味着相同字段不能生成两次