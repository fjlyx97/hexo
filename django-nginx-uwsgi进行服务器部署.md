---
title: django+nginx+uwsgi进行服务器部署
date: 2018-06-08 13:37:37
tags:
- Linux
- 学习
- 服务器
categories : "python"
---

> - django的服务器配置可以说是十分繁琐的部分
> - 哪怕严格按照步骤也会有不明错误
> - 再此详细记录操作步骤
> - 版本Ubuntu 16.04 LTS 64位
> - 服务器：腾讯云学生机

<!--more-->

> - 由于配置繁琐，所以操作步骤实在全新系统进行配置

# 准备django文件
- 由于已经有过django项目，所以直接将django上传至服务器
- 目录树如下：
```shell
ubuntu@VM-246-186-ubuntu:~/site$ tree Blog
Blog
├── Blog
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── settings.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── wsgi.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── db.sqlite3
├── manage.py
├── myblog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── __init__.py
│   │   └── __pycache__
│   │       ├── 0001_initial.cpython-36.pyc
│   │       └── __init__.cpython-36.pyc
│   ├── models.py
│   ├── __pycache__
│   │   ├── admin.cpython-36.pyc
│   │   ├── __init__.cpython-36.pyc
│   │   ├── models.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── views.cpython-36.pyc
│   ├── static
│   │   └── myblog
│   │       ├── css
│   │       │   ├── semantic.css
│   │       │   └── themes
│   │       │       └── default
│   │       │           └── assets
│   │       │               ├── fonts
│   │       │               │   ├── brand-icons.eot
│   │       │               │   ├── brand-icons.svg
│   │       │               │   ├── brand-icons.ttf
│   │       │               │   ├── brand-icons.woff
│   │       │               │   ├── brand-icons.woff2
│   │       │               │   ├── icons.eot
│   │       │               │   ├── icons.otf
│   │       │               │   ├── icons.svg
│   │       │               │   ├── icons.ttf
│   │       │               │   ├── icons.woff
│   │       │               │   ├── icons.woff2
│   │       │               │   ├── outline-icons.eot
│   │       │               │   ├── outline-icons.svg
│   │       │               │   ├── outline-icons.ttf
│   │       │               │   ├── outline-icons.woff
│   │       │               │   └── outline-icons.woff2
│   │       │               └── images
│   │       │                   └── flags.png
│   │       └── js
│   │           ├── jquery-3.3.1.js
│   │           ├── jquery-3.3.1.min.js
│   │           ├── semantic.js
│   │           └── vue.js
│   ├── tests.py
│   ├── urls.py
│   └── views.py
└── templates
    └── myblog
        ├── detail.html
        └── index.html
```

- 目录树不必在意，主要操作的文件为manage.py

# 配置环境
## 安装依赖
- Ubuntu系统默认为python2，但是这里测试的django版本为2.0以上，需要python3版本来安装
- 为了防止打乱系统环境，还需要virtualenv进行管理python环境
```
sudo apt-get install python3-pip
sudo pip install virtualenv
```
- 准备就绪之后进行创建新环境
```python
virtualenv env27 --python=python3.5
#其中env27为环境名称，自己替换
```
- 进入新环境
```
source test/bin/activate
```
- 由于我的django项目中包含了django-suit以及markdown，所以在运行django前需要提前准备完这两个包
```python
(test) ubuntu@VM-246-186-ubuntu:~/site$ pip install django django-suit markdown
```

## 进行django测试
- 在云服务器上部署确保安全组端口已经开放
- 确保是在虚拟环境当中，注意命令行前的test
### 配置setting文件
- 首先要让服务器知道静态文件的位置以及允许访问的ip
- 进入Blog目录中，编辑setting.py文件
```python
#找到ALLOWED_HOSTS = []
#将其配置为
ALLOWED_HOSTS = ['*']
#配置静态文件目录，在文件尾部直接添加
STATIC_ROOT = os.path.join(BASE_DIR , "static/")
#找到DEBUG = True，改为False，亦可最后再来改动方便定位错误
DEBUG = False
```
- 配置完毕之后，回到根目录执行收集静态文件操作
```
python manage.py collectstatic
```
- 可以看到在目录当中生成了一个static的文件夹，该文件夹包含了网站的全部静态文件


## 登录网站测试
- 在项目根目录处运行服务器测试命令，确保8001端口未被占用，如果被占用需手动指定端口号
```
python manage.py runserver 0.0.0.0 8001
```
- 如果一切顺利的话，应该可以通过IP进行直接访问网站
- 在浏览器输入IP地址+端口号进行服务器测试
- 可以观察到网站可以进入，并且CSS样式加载正常

# 安装uwsgi
- 由于前面环境已经配置妥当，安装uwsgi只需要一句命令，请确保在虚拟环境当中
```
pip install uwsgi
```
- 在网站根目录下执行
```
uwsgi --http :8001 --plugin python --module Blog.wsgi
# 其中Blog.wsgi需要自行替换成自己的网站名字（使用startproject创建的网站名），比如自己的网站为Test，则修改为Test.wsgi
```
- 通过访问IP地址加端口号进行测试，例如118.25.3.239:8001（例子虚拟，自行替换IP）
- 如果访问正常我们应该看到的是不带CSS样式的界面

## 配置uwsgi文件
- 为了让服务器可以运行，我们需要手动在根目录创建一个uwsgi.ini的文件
- 内容如下：
```
[uwsgi]
#chidr用于指定自己的网站根目录（自行更改）
chdir =  /home/ubuntu/Blog
#module指定网站中APP的文件（自行更改）
module = Blog.wsgi
#home用于指定python的虚拟环境，即我们最初创建的虚拟环境位置（自行更改）
home = /home/ubuntu/env27
#master不用更改
master = true
#socket用于指定端口号，可以不更改
socket = :8001
#以下两个可以不用更改
chmod-socket = 666
vacuum = true
```
- 配置完内容之后可以在shell中运行，并访问网站观察十分成功运行
```
sudo uwsgi --ini uwsgi.ini
```

# 配置nginx
- 安装过程很简单，一句命令
```
sudo apt-get install nginx
```
- nginx安装完默认是启动的，可以通过直接在浏览器访问ip的方式观察到nginx的欢迎界面
> - 这里我原先使用的Chrome浏览器，不知为何会强制重定向到https，导致网站无法访问
> - 更换浏览器以后访问便正常了
> - 推荐使用edge或者自带的ie进行简单的测试 

- nginx默认会读取/etc/nginx/sites-enabled/default文件中的配置，修改其中配置如下：
```
upstream django {
        server 127.0.0.1:8001; #此处端口可以替换，需与uwsgi.ini之中的端口一致
}
server {
        listen 80;
        server_name 118.25.3.239; #此处需要自行替换成自己的IP地址
        charset utf-8;
        client_max_body_size 75M;
        #django配置文件
        location /static {
                alias /home/ubuntu/Blog/static; #此处需要自行替换为网站根目录下的静态文件夹
        }
        #以下不必改动
        location / {
                uwsgi_pass django;
                include /etc/nginx/uwsgi_params;
        }
}
```

## 重启服务器
- 一切配置完毕之后，执行命令
```
sudo /etc/init.d/nginx restart
```
- 浏览器中输入IP即可发现网站成功构建完毕

# 参考文章及视频
## 文章
- 很详细的一篇文章
- [Ubuntu上通过nginx部署Django笔记](https://www.cnblogs.com/jhao/p/6071790.html)
## 视频
- 视频讲解非常透彻，如果一步一步跟着执行出错可能性极低
- [【教程】Nginx + uWsgi 部署 Django + Mezzanine 生产服务器(「自由小径」自由软件教程) - 1/2](https://www.bilibili.com/video/av10247256?t=1838)
- [【教程】Nginx + uWsgi 部署 Django + Mezzanine 生产服务器(「自由小径」自由软件教程) - 2/2](https://www.bilibili.com/video/av10244432?t=996)

# 2019.5.17后续更新
1. 近期在部署时发现，如果在自己的程序当中，使用了python的多线程，需要在uwsgi.ini文件当中加入enable-threads=True参数
2. 在部署的时候，使用uwsgi --ini读取配置的时候，出现了无法访问的情况，并且报出如下错误
> - invalid request block size: 21573 (max 4096)...skip

因此我们在测试时，需要修改配置文件中的socket为http，正式部署到nginx时再修改回socket，如下
```
[uwsgi]
...省略
socket = :8001 => http = :8001
...省略
```