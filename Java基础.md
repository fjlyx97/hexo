---
title: Java基础
date: 2021-04-20 11:26:14
tags:
- 编程
- java
categories : "java家族"
---

> - Java入门基础
> - 从零开始学习Java

<!--more-->

# 基础
1. Java文件名必须和public class名称一致
2. Java文件可以有多个class，public class**只能有一个**
3. /** */ Java文档注释，可以使用JavaDoc生成文档
4. luyten java反编译工具
5. long类型最好在末尾增加L，好区分
6. float类型有精度，**只能在小数点后七位**，必要使用BigDecimal
7. final关键字代表常量获最终变量，不可修改。
8. 数组属于引用类型，创建完成数组后，默认会初始化。数组变量是在栈上，指向堆上的数据。
9. 成员变量有初始值，局部变量没有

# 引用类型
Java中，对象和数组，都是通过引用进行操作的

# 类
1. 默认有无参构造函数。
2. 如果定义了有参构造函数，无参构造函数就失效。
3. 可以定义多个有参构造函数
4. 当构造方法中，需要调用其他构造方法时，可以直接使用**this()**，但是必须位于第一行
5. 只能单继承

## 访问控制
Java有成员变量四种访问控制权限：public,protected,default,private
- public: 所有都可以访问
- protected:当前类、当前包、子类
- default: 当前类，当前包
- private： 当前类

Java类有两种访问权限：public,default
- public:同一个项目所有类
- default:被同一个包中所有类

## static关键字
- 类中定义的static变量，所有类成员共享
- 位于方法区的静态存储区
- 一般工具类方法定义成static

# 代码块
静态代码块->构造代码块->普通代码块
## 构造代码块
```Java
class App() {
    {
        System.out.println("构造代码块");
    }
}
```
构造代码块会被添加到构造函数的开头，因此先执行。

## static代码块
数据库连接等其他需要提前准备好的代码，会放在static代码块中，只会执行一次
```Java
class App() {
    static {
        System.out.println("构造代码块");
    }
}
// 多次载入这个类，只会执行一次
```

## 同步代码块
多线程会使用，用来给共享空间加锁

# package
- 解决类重名问题
- 方便管理类
- 如果导入了两个同名的包，只能通过package+完整名称识别
- import static .... 静态导包，导入所有静态方法

## 常见的包
- java.lang Java语言核心类，提供String,Math,Integer,System,Thread等常用功能（不需要手动导入）
- java.awt 主要用于构造gui
- net 网络包
- util 工具包
- io 输入输出流