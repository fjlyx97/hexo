---
title: Nginx笔记
date: 2020-08-08 20:25:10
tags:
- 学习
- Linux
- 服务器
categories : "Linux"
---

> - Nginx核心功能
> - 记录学习笔记

<!-- more-->

# Nginx命令
- 查看命令：nginx -h
- 启动命令： nginx -c file
- 停止命令： nginx -c stop/quit
- 测试配置文件命令： nginx -tq
- 平滑重启命令： nginx -s reload

# Nginx请求定位
当location出现多个表达式，并且URL同时匹配上了多个Location，会出现404的问题