---
title: archlinux下的QQ配置
date: 2018-06-17 21:47:21
tags:
- Linux
- 学习
- 服务器
categories : "linux"
---

> - QQ重度患者
> - Linux配置QQ过程繁琐，坑众多
> - 在此记录Linux下的QQ配置
> - 发行版：archlinux 64位

<!--more-->

# 安装wine
- 由于是基于archlinux，所以安装配置十分简单
- 首先先启用Multilib仓库
```bash
sudo vim /etc/pacman.conf
#取消以下注释
[multilib]
Include = /etc/pacman.d/mirrorlist
```
- 更新系统
```bash
sudo pacman -Syy
```
- 安装wine
```
sudo pacman -S wine wine_gecko wine-mono winetricks
```

# wine的版本问题
- 最新的wine版本似乎不支持直接使用
- 可以在官方wiki找到如下：
> - Archlinux打包的wine-3.8与wine-3.9.1版本均在安装QQ后无法打开，出现无法加载gdi32.dll的错误
> - 可以使用命令“sudo pacman -U /var/cache/pacman/pkg/wine-3.7-1-x86_64.pkg.tar.xz”将wine降级到之前的3.7版本解决。
> - 为了避免被升级到新3.9版本而无法使用QQ，可以编辑"/etc/pacman.conf"文档，去掉"IgnorePkg"一行前面的"#"号，并在该行"="号后面添加"wine"；在后续更新中该问题得到解决后，记得去掉wine，以便更新之。
- 可以很明显观察到如果想要使用QQ，必须将wine降级到3.7
- 所以需自行下载wine3.7的安装包手动降级
- 找到下载地址：[wine3.7]("https://pan.baidu.com/s/1IAUgoAMlmmcQqDMeTE9P_w")
```bash
sudo pacman -U wine-3.7-1-x86_64.pkg.tar.x
```

# 安装QQ
- 由于目前许多QQ都十分不稳定
- 官方wiki中的deepin-qq-im在这里安装完后无法使用，谷歌许久后，选择清风QQ进行安装
- 地址如下：[清风QQ](" wine-3.7-1-x86_64.pkg.tar.xz")
- 下载完其中压缩包之后，按说明的命令进行解压
```bash
tar xvf wineQQ9.0.3_23719.tar.xz -C ~/
```
- 解压完之后即可在程序中看到QQ的图标

# 已知问题
- 无法记住密码
- 无法打开群文件
- 无法使用我的手机进行传输文件

# 删除方法
```bash
rm -rf ~/.wine
rm -rf ~/.local/share/applications/wine-QQ.desktop
rm -rf ~/.local/share/icons/hicolor/256x256/apps/QQ.png
```

# 参考文章
[清风的网络空间]("https://phpcj.org/wineqq/")
[ArchLinux安装配置WineQQ]("https://whoisnian.com/2017/07/23/ArchLinux%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEWineQQ/")