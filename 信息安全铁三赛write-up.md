---
title: 信息安全铁三赛write_up
date: 2018-01-19 09:13:45
tags:
- 信息安全
- write_up
- 渗透
categories: 信息安全
---

>- 信息安全铁三赛write_up
>- 题目共有六题
>- 随手记录

<!--more-->

# 权限

>- 题目描述：你是管理员吗？
>- 解题链接：http://ctf4.shiyanbar.com/web/root/index.php
>- 答案格式：flag{}

## 步骤
- 首先先浏览网页源代码，易于发现在源代码底部有提示如下：
```html
<!--password.txt>
```
- 可以发现存在password.txt，在地址栏构造出地址如下：
```html
http://ctf4.shiyanbar.com/web/root/password.txt
```
- 可以进入包含一个许多密码的页面，由此可以推测出密码是其中的一个。
- 密码框长度默认只有5个长度，手动用审查元素找到**maxlength=5**，将数据改成999
- 由于密码过多，我们可以需要使用buresuite进行爆破，使用教程可以参考:[使用教程](https://www.cnblogs.com/nieliangcai/p/6689915.html
)
- 通过爆破以后，我们可以发现**Nsf0cuS**这个密码有些与众不同。它的提示是**这里没有flag哦！**
- 再次抓包发现Response当中有一项：
```
Set-Cookie: newpage=MjkwYmNhNzBjN2RhZTkzZGI2NjQ0ZmEwMGI5ZDgzYjkucGhw;
```
- 使用base64解码后得到一个php文件，如下：
```
290bca70c7dae93db6644fa00b9d83b9.php
```
- 构造URL后可以看到小黑留言板，再次抓包尝试
- 通过分析发现Cookie中有个**IsLogin=0**这个参数，并且**userlevel**为guest
- 将IsLogin改为1，userlevel改为root，点击go再次尝试
- 可以得到flag为：
```
Flag=flag%7BC0ngratulati0n%7D
```
- 使用url解码得到flag为
```
Flag=flag{C0ngratulati0n}
```

# IOS

>- 题目描述：无
>- 解题链接：http://ctf4.shiyanbar.com/web/IOS/index.php
>- 答案格式：flag{}

## 步骤
- 通过界面可得系统已经升级到ios99，可以通过抓包修改user-agent来更改所需系统
- user-agent如下：
```
User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 99 like Mac OS X) AppleWebKit/534.46 (KHTML, like Gecko) Version/5.1 Mobile/9A5313e Safari/7534.48.3
```
- 发送后得到flag为：
```
flag{LMvBi8w9$m1TrgK4} 
```

# 照猫画虎

>- 题目描述：照猫画虎
>- 解题链接：http://ctf4.shiyanbar.com/web/copy/index.php
>- 答案格式：flag{}

## 步骤
- 通过界面可知**1234567890th**的访客可以得到奖励，当前是**2699th**的访客
- 抓包分析，发现Request中存在如下：
```
Visitor=MjY5OTowNDE1NzQwZWFhNGQ5ZGVjYmM4ZGEwMDFkM2ZkODA1Zg%3D%3D;
```
- 先使用url解码，再使用base64解码发现如下：
```
2699:0415740eaa4d9decbc8da001d3fd805f
```
- 分析发现冒号后面可以使用MD5解密，解密得到答案为**2699**，也就是最终结果为：
```
2699:2699
```
- 使用1234567890逆向加密回去，可得
```
1234567890:e807f1fcf82d132f9bb018ca6738a19f
```
- 再使用base64和url加密，可得
```
MTIzNDU2Nzg5MDplODA3ZjFmY2Y4MmQxMzJmOWJiMDE4Y2E2NzM4YTE5Zg%3D%3D
```
- 使用buresuite改变**Visitor**的值为如上密码，发送后可得flag为：
```
flag{T4mmL9GhpaKWunPE}
```

# 问题就在这

>- 题目描述：找答案 GPG key: GhairfAvvewvukDetolicDer-OcNayd#
>- 解题链接：http://ctf4.shiyanbar.com/ste/gpg/john.tar.gz.gpg
>- 答案格式：flag{}

## 步骤
- 首先下载下来解题文件，由题意可得这个文件通过GPG加密，并且密钥已经给出
- 打开**gpg4win**这个软件，使用如上密钥，可以解密出文件**john.tar.gz**，解压后得到**john.tar**，发现还是个压缩包，再次解压可以得到**john-in-the-middle.pcap**这个数据包文件。
- 使用WireShark打开这个数据包，观察到其中有许多PNG图片，点击上方文件，导出对象，HTTP来导出所有文件，得到一堆文件
- 使用**Stegsolve**来逐个分析图片文件，可得到flag为：
```
flag{J0hn_th3_Sn1ff3r}
```

# 你最美

>- 题目描述：无
>- 解题链接：http://ctf4.shiyanbar.com/misc/123/123.exe
>- 答案格式：flag{}

## 步骤

- 首先下载题目文件，并拖入winhex进行分析，发现它是一个png图片，并使用base64加密。
- 使用在线图片转换base64工具，将winhex的内容全部复制进去，可以还原出一个二维码，扫描后可得flag为：
```
flag{you are beautiful}
```

# shellcode

>- 题目描述：无
>- 解题链接：http://ctf4.shiyanbar.com/re/shellcode/shellcode.txt
>- 答案格式：flag{}

## 步骤

- 首先打开题目所给的链接，发现如下字符串:
```
\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x34\x00\x00\x00\x65\x63\x68\x6f\x20\x5a\x6d\x78\x68\x5a\x33\x74\x54\x53\x45\x56\x73\x62\x47\x4e\x76\x5a\x47\x56\x66\x53\x56\x4e\x66\x63\x32\x39\x66\x51\x32\x39\x76\x62\x48\x30\x4b\x7c\x62\x61\x73\x65\x36\x34\x20\x2d\x64\x00\x57\x53\x89\xe1\xcd\x80
```
- 使用chrome的控制台，使用如下语句
```js
alert("\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x34\x00\x00\x00\x65\x63\x68\x6f\x20\x5a\x6d\x78\x68\x5a\x33\x74\x54\x53\x45\x56\x73\x62\x47\x4e\x76\x5a\x47\x56\x66\x53\x56\x4e\x66\x63\x32\x39\x66\x51\x32\x39\x76\x62\x48\x30\x4b\x7c\x62\x61\x73\x65\x36\x34\x20\x2d\x64\x00\x57\x53\x89\xe1\xcd\x80")
```
- 可以得到字符串如下：
```
jXRfh-cçh/sh h/binãRè4  echo ZmxhZ3tTSEVsbGNvZGVfSVNfc29fQ29vbH0K|base64 -d WSáÍ
```
- 分析可得下面代码是使用base64位加密的：
```
ZmxhZ3tTSEVsbGNvZGVfSVNfc29fQ29vbH0K
```
- 解密后可得flag为：
```
flag{SHEllcode_IS_so_Cool}
```