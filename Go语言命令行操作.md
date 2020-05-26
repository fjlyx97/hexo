---
title: Go语言命令行操作
date: 2020-05-24 10:50:05
tags:
- 编程
- 学习
- golang家族
categories : "golang家族"
---

> - Go语言命令行操作
> - 基于Github专栏:Go命令教程
> - 记录自己常用的命令

<!--more-->

[Go命令教程]("https://hyper0x.github.io/go_command_tutorial/#/")

# GO工具目录存放位置
**$GOROOT/pkg/tool/<平台相关目录>**

# 源码文件类别
- 命令源码文件（总是作为程序入口）
- 库源码文件（存放接口，全局变量等）
- 测试源码文件

# go build
Go build后参数如果不跟代码包名，就会编译当前目录所对应的代码包。

- go build命令在编译只包含库源码文件的代码包（或者同时编译多个代码包）时，只会做检查性的编译，而不会输出任何结果文件。（不包含main）
- 编译时如果两个代码包都含main，会报重定义错误

## -v参数
这个参数会将执行过程中构建的包名打印出来
```
$ go build -v
go: downloading github.com/fjlyx97/golang-stress-tool/stress_core v0.0.0-20200520014334-e5bde455f894
github.com/fjlyx97/golang-stress-tool/stress_core
golang-stress
```

## -o参数
指定输出的文件名

## -a参数
强行对代码包进行构建，哪怕已经是最新的了

# go install
编译并安装指定的代码包及它们的依赖包。相比go build，go install多做了一件事：安装编译后的结果文件到指定目录。参数和go build类似
- 比如我电脑当中，存放的安装路径为：C:\Users\Administrator\go\bin，前缀为GOPATH

# go get
这个命令主要用于下载源码包，并且保存的路径为GOPATH中包含的第一个工作区。
- 如从github下载的包：C:\Users\Administrator\go\pkg\mod\github.com\

go get可以接受go build和go install中所有可接受的参数，因为go get包含了编译和安装两个动作

## -d参数
只下载不安装

## -u参数 -f参数
-u用于强行更新代码包，-f参数一般和-u结合，用于忽略对已下载代码包的导入路径检查（适用fork别人的项目）

## -insecure
允许http连接下载

## -fix
使用这个参数，可以修复因版本不同带来的代码规范差异，如从go0.9->go1.5

# go clean
这个命令用于删除执行其他命令产生的一些文件和目录

## -x参数
打印出命令的执行过程

## -modcache（常用）
清空mod下载产生的缓存

# go run
编译并运行命令源码文件（main），并且只接受Go源码文件作为参数

# go test
用于对Go编写的程序进行测试，测试源码文件必须以**_test.go**结尾

# go list
列出指定的代码包信息

## -json标记
可以看到错误信息

## -m参数
列出的是模块而不是包

## go list -m json all
Goland自动运行的命令，

# go fmt
按GO语言规范，格式化当前模块中的所有Go代码。可以看成是gofmt命令的简单封装

# go env
列出go环境，在公司中，由于是内网git，并且需要开启module模块，所以需要额外设置：
```
set GO111MODULE=on                             //开启module
set GONOPROXY=                                 //如果是内网的话，就不走GOPROXY的代理
set GONOSUMDB=                                 //内网不校验GO模块
set GOPROXY=https://goproxy.cn,direct          //国内设置代理源，加速下载
set GOSUMDB=sum.golang.org                     //校验模块，保证下载的包是安全的，和大多数人用的是一致的
```