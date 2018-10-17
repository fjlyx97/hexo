---
title: web基础知识
date: 2017-10-21 13:42:24
tags:
- 安全
- 学习
- web
categories : "web安全笔记"
---

> - 在不断打代码的过程中，消遣生活
> - 丰富课余生活
> - 记录web安全学习之路
> - 根据网易web安全工程师课程所作笔记
> - 本文为第一章
> - 大坑已填
<!--more-->

# web是什么？
- World Wide Web 万维网
- 非常普遍的互联网应用
- 平时流量的网页
- 经历了Web1.0，Web2.0等阶段
  > - Web1.0主要是静态页面
  > - Web2.0类似于博客等页面

# web的工作流程
- 通过dns请求，发送域名到dns服务器，得到解析后的ip地址，然后便可以通过解析完毕的ip地址来访问一个网址。

# url的详细格式
- schema://host[:port#]/path/.../[?query-string][#anchor]
  > - schema：底层协议，如:http,https,ftp
  > - host：服务器域名或者IP地址
  > - port： 端口，默认80端口
  > - path： 访问资源的路径
  > - ?query-string： 发送给http服务器的数据
  > - anchor： 锚

# 什么是HTTP?
- web使用的协议，即超文本传输协议
- 应用最广泛，web基础
- Hyper Text Transfer Protocol
## 什么是HTTPS?
- 可以简单来说，HTTPS = HTTP + SSL
- 相对于HTTP来说，HTTPS更为安全
- HTTP标准端口是80，HTTPS标准端口则为443
- HTTP无需加密，HTTPS对传输的数据进行加密
- HTTP无需证书，HTTPS需要认证证书


# Javascript Bom
- 警告弹窗 alert()
- 确认弹窗 confirm()
- 提示弹窗 prompt()
  > - Bom浏览器对象模型
  > - 以后测试可能会用到
  > - 检验是否有漏洞
  > - 建议熟记

# HTML常用标签
```html
<h1></h1> 标题标签，最多到<h6></h6>，标题依次从大小
<p></p>   段落标签，浏览器会自动已排版的格式
<a href="https://www.baidu.com">用了href属性</a>    链接标签
<img src="path" width=数字 height=数字 />   插入图片的标签
<!--注释-->   注释标签
<hr />        插入水平线
<br>          换行标签
<b>           文本加粗标签
<strong>      文本加粗标签
<big>         文本放大标签
<em>          文本斜体标签
<small>       文本缩小标签
<sub></sub>   文本下标标签
<sup></sup>   文本上标标签
<del></del>   文本删除线标签
<ins></ins>   文本下划线标签
<pre></pre>   文本格式化标签，允许控制空行和空格
<a href="mailto:webmaster@example.com">Mr L</a> 邮箱标签

<!--大坑待填-->
```

# 自定义本地host
- windows系统下，hosts文件默认在：C:\Windows\System32\drivers\etc目录下
- linux系统下，hosts文件则在 /etc/hosts
  > - 可以通过输入IP + 网址来解析 ， 如下
  > - 192.168.1.138 www.testweb.com

# Mysql操作
- SQL对大小写不敏感，分号代表语句结束
```sql
CREATE DATABASE 数据库名字; #创建数据库
show databases;            #查看数据库
USE 数据库名字;             #使用数据库
DROP DATABASE 数据库名字;   #删除数据库
SELECT database();         #打印当前数据库名称
SELECT current_user();     #打印当前登录用户名
SELECT load_file(path);    #打开文件
----------------分割线-----------------
/*创建数据表*/
CREATE TABLE 表名{id int(4), name char(20), sex char(20)};
/*查看数据表*/
show tables;
/*查询数据表*/
SELECT * FROM 表名; #后面可选WHERE id = 1 and name = "LY";等类似句子 
/*插入数据表*/
INSERT INTO 表名(name,sex) VALUES("大风烈酒孤独自由","Malo"); #id可以自增长
/*更新数据*/
UPDATE 表名 SET name = "LY" WHERE id = 1;
/*删除数据*/
DELETE FROM 表名 WHERE name = "LY";

#大坑待填
```