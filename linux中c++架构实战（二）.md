---
title: linux中c++架构实战（二）
date: 2019-01-22 20:31:17
tags:
- linux
- 学习
- c家族
categories : "Linux"
---

> - linux c++架构实战系列课程笔记（二）
> - 服务器框架初步

<!-- more-->

# 服务器框架初步
## 目录结构规划
```
- nginx：主目录
- _include目录： 专门存放各种头文件
- app目录： 放主应用程序.c，如main函数以及比较核心的目录.
    link_obj：临时目录，存放临时的.o文件，不手工创建，后续使用makefile脚本创建
    dep:临时目录，会存放临时的.d开头的依赖文件，依赖文件能告知系统那些相关的文件发生变化，需要重新编译
    nginx.c：主文件，main()入口函数
    ngx_conf.c，普通源码文件
- misc目录：专门存放杂合性不好归类的C文件
- net目录：专门存放和网络处理相关的文件
- proc目录：专门存放和进程处理有关的多个文件
- signal目录：专门用于存放有关信号处理的文件
    ngx_signal.c
```

## Makefile编写
- makefile:编译项目的入口脚本，起到总体控制
- config.mk：配置脚本，应付一些可变的东西
- common.mk 最河西的编译脚本，定义makefile的编译规则，依赖规则等，通用性很强，各个子目录中都用到这个脚本
