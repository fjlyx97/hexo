---
title: docker使用笔记
date: 2018-12-27 21:44:23
tags:
- linux
- 学习
- docker
categories : "Linux"
---

> - docker在Ubuntu下的安装使用
> - docker的使用命令

<!-- more-->

# 启动docker后台服务
```
sudo service docer start
```
- 测试hello-world
```
sudo docker run hello-world
```

# docker常用命令
## 启动docker
- -d参数：后台运行
- -i参数：交互式运行
- -t参数：分配一个伪输入终端，一般和-i使用
- -p参数：端口映射 本地:容器
- --name参数： 指定容器姓名
```
docker run -d -it 容器名
```

## 查看本地镜像
查看本地镜像，-a参数为列出所有
```
docker images -a 
```

## 删除镜像
- 使用-f参数强制删除
```
docker rmi 镜像名
```

## 查看正在运行的docker
- -a参数可以查看历史的记录
```
docker ps
```

## 重命名docker
```
docker rename 索引名 新名字
```

## 删除docker
```
docker rm 索引
```

## 重新启动一个docker（已存在）
```
docker start 索引
```

## 进入一个docker
```
docker attach 索引
docker exic -it 索引 /bin/bash #退出时容器不退出，或者CTRL+P+Q
```

## 查询docker
```
docker search 名字
```

## 拉取docker
```
docker pull 名字
```