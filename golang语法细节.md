---
title: golang语法细节
date: 2020-05-16 16:50:05
tags:
- 编程
- 学习
- golang家族
categories : "golang家族"
---

> - 腾讯云后端使用编程语言为Golang
> - 记录学习golang的细枝末节

<!--more-->

# Init函数
有个特殊的函数，func init() {} ，会在被导入的时候执行，先于main函数

# 类型转换
所有隐形类型转换都不允许，必须显式写出，如type(转换对象)

- 即使使用了type FI int进行申明，FI和int也不是一个类型，需要强制转换

# iota
这个值用于枚举，如下代码：
```golang
const (
	success = iota //0
    fail          //1
    timeout        //2
)
```

# 变量初始化
在golang当中，所有内存都会被初始化，如0,"",nil,false

# 引用类型
Go 语言中，指针属于引用类型，其它的引用类型还包括slices，maps，interface和channel。被引用的变量会存储在堆中，以便进行垃圾回收，且比栈拥有更大的内存空间。（重要：数组不是引用类型，因此传入函数时会发生拷贝）

# 使用:=赋值操作符
:=只能用于函数体内，不能用于全局变量的赋值，且要确定这个是不存在的。

# 指针
不能对一个const或者一个常量取地址

# 可变长参数
如果一个函数当中出现如func f(a int ,b ...int) {} ， 则b传入之后为一个（必须是）slice，并且我们可以通过如下方法传入参数：
```golang
b := []int{1,2,3,4,5}
var c int = 10
f(c,b...)
```

# 数组
对于golang的数组来说，它的大小一定是一个常量值，并且array和string不一样，array的索引下标可以改变值，但是string不允许改变。并且允许编译器自动计算大小，需要填入...，如nums := [...]int{1,2,3}