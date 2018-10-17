---
title: web安全工具
date: 2017-10-23 17:00:20
tags:
- 安全
- 学习
- web
categories : "web安全笔记"
---

> - 根据网易web安全工程师所做笔记
> - 记录学习之路
> - 前文完结
> - 本文为第三章
> - 大坑待填

<!--more-->

# 浏览器拓展初级
- 可以将阻止弹出式窗口关闭，便于测试

## Firefox四个插件
- Firebug
- HackBar
- Advanced Cookie Manager
- Proxy Switcher

#代理的工作原理
- "代理"可对经过的数据包进行记录，拦截，修改，再次发送，丢弃等操作

##浏览器的代理设置
- 可以手动设置
- 常用HTTP代理工具：Burpsuite ， Charles ， Fiddler

#敏感文件探测实践
- 猜测文件名，根据返回的HTTP状态码判断文件是否存在
- 200：文件存在
- 301：文件发生跳转
- 用"御剑"扫描敏感文件
- 通过判断index的文件类型确定网站脚本类型，如在网址中访问index.asp、index.php、index.aspx、index.jsp来选择字典