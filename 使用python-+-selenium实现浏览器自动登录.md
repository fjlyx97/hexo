---
title: 使用python + selenium实现浏览器自动登录 
date: 2017-08-28 17:02:31
tags: 
- python
- 编程
- 学习
categories: "python记录"
---
> - 使用python + selenium实现浏览器自动登录
> - python版本为3.6.2
> - 系统为windows10
> - 记录自己学习的心得，也是为了防止以后时间久了自己也遗忘了

<!--more-->
# 1.安装selinium
  安装的过程十分简单，只需要使用**pip**安装即可
```
pip install selenium
```
# 2.安装对应的webdriver（驱动）
本文以edge浏览器为例子
百度edge webdriver，搜索完第一个就是微软的官网了。
这里附上链接:[WebDriver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/)
如下图：
![WebDriver.png](http://upload-images.jianshu.io/upload_images/2692660-8651f559291c75c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择对应的**版本**,**这里十分重要!!**
版本可以通过windows的运行快速查看
```
dxdiag
```
如下图：

![Dxdiag.png](http://upload-images.jianshu.io/upload_images/2692660-c9affc27c1e378a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下载完毕之后，将**MicrosoftWebDriver.exe**复制到**python**安装目录下的**scripts文件夹**里面。至此安装环境配置完毕，开始实战。
# 3.实战
- 首先在python中导入selenium的webdriver的库
`from selenium import webdriver`
- 实例化出一个浏览器
`browser = webdriver.Edge()`
- 打开网址需要使用get方法，我们在这里以小米官网为例子
`browser.get("https://account.xiaomi.com/pass/serviceLogin?callback=https%3A%2F%2Forder.mi.com%2Flogin%2Fcallback%3Ffollowup%3Dhttps%253A%252F%252Fwww.mi.com%252F%26sign%3DNzY3MDk1YzczNmUwMGM4ODAxOWE0NjRiNTU5ZGQyMzFhYjFmOGU0Nw%2C%2C&sid=mi_eshop&_bannerBiz=mistore&_qrsize=180")`
我们可以试着运行代码，可以看到浏览器已经自动打开小米官网了
- 获取元素
元素的方法有很多种：
> find_element_by_id                                             
find_element_by_name                                           
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector
>
在这方面网上有很多资料
首先我们要定位到一个元素，在这里我们可以用浏览器自带的工具
右键登录按钮，点击**检查元素**，如图：
![小米官网](http://upload-images.jianshu.io/upload_images/2692660-17b2ba204988baeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们就可以定位到元素了

![登录](http://upload-images.jianshu.io/upload_images/2692660-5e8e415572bb8acd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以查看到他有着**name**属性，**id**属性
**定位元素的方法多种多样，没有规定一定要用哪一种，适合即可。**
```python
name = browser.find_element_by_name("user")
#亦或者
name = browser.find_element_by_id("username")
```
- 于是乎，我们已经定位到账号的输入框了。
接下来是输入内容，输入内容可以使用**send_keys**的方法
`name.send_keys("hello")`
运行试试看，我们会发现已经成功在输入框中输入我们所要的内容。

![hello](http://upload-images.jianshu.io/upload_images/2692660-d1bbce6cafcbb70c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 同理我们可以定位到密码框
`passwd =  browser.find_element_by_name("pwd")`
- 以及往里面填充内容
`passwd.send_keys("hello")`
如下图所示：

![登录](http://upload-images.jianshu.io/upload_images/2692660-38abe558858162f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 输入完账号密码，最后一步便是点击登录按钮了
```python
login_button = browser.find_element_by_id("login-button")
login_button.click()
```
- 运行试试，我们可以看到已经登录过了，当然账号密码是随便输入的。

![image.png](http://upload-images.jianshu.io/upload_images/2692660-7d61525737a00cae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后附上代码：
```python
from selenium import webdriver
import time

browser = webdriver.Edge()
browser.get("https://account.xiaomi.com/pass/serviceLogin?callback=https%3A%2F%2Forder.mi.com%2Flogin%2Fcallback%3Ffollowup%3Dhttps%253A%252F%252Fwww.mi.com%252F%26sign%3DNzY3MDk1YzczNmUwMGM4ODAxOWE0NjRiNTU5ZGQyMzFhYjFmOGU0Nw%2C%2C&sid=mi_eshop&_bannerBiz=mistore&_qrsize=180")
time.sleep(2)
name = browser.find_element_by_name("user")
name.send_keys("账号")
passwd = browser.find_element_by_name("pwd")
passwd.send_keys("密码")
login_button = browser.find_element_by_id("login-button")
login_button.click()
```
# 4.其它定位
## xpath定位
- 通常情况下，我们不一定能通过**name**或**id**定位,这时候我们可以用其它方法，通过**xpath**就是一种不错的选择，这里以百度的输入框为例。
xpath可以通过谷歌浏览器或者其它浏览器直接查看，如图：

![xpath](http://upload-images.jianshu.io/upload_images/2692660-85ac56693710d953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 单击后即可复制,如下
`//*[@id="kw"]`
- 我们可以使用它
`path = browser.find_element_by_xpath("//*[@id='kw']")`
- **记得将里面的双隐号改成单隐号，否则报错。**
可以试试是否定位到了
`path.send_keys("hello")`
***
## 使用elements定位
很多时候元素定位都很模糊，要定位到并不是十分容易，我们可以换个思路，同时定位一组元素，通过取下标的方法，读取元素，以百度的首页为例：

![首页](http://upload-images.jianshu.io/upload_images/2692660-550f034798e9ef3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 通过查看元素我们可以发现：

![查看元素](http://upload-images.jianshu.io/upload_images/2692660-6a33da589c485855.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 上面一组元素具有许多相同的特征，于是我们可以使用如下代码：
```python
elements = browser.find_elements("css selector" , ".mnav")
#或是
elements = browser.find_elements_by_css_selector(".mnav")
```
两个定位是相同的，不过写法不同而已，这个方法和之前的区别主要就是find_**elements**，使用的是**复数**定位，最后通过**下标**来选择元素。
```python
elements[0].click()
```
这样便是点击第一个元素，也就是新闻的按钮。
附上测试代码:

![测试](http://upload-images.jianshu.io/upload_images/2692660-6f0d0010b8ff4e85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 5.终

>- selenium的资料网上很多，遇到问题善用搜索引擎，一般问题都可以得到解决。
>- # 大坑
> 这里要特别说明一点，在测试时，浏览器的缩放比例**不能调，不能调，不能调。**这真的是一个大坑，我当时电脑的缩放比例是**125%**,定位元素始终不能成功，一直会提示：
![提示.png](http://upload-images.jianshu.io/upload_images/2692660-1e0138d8e151b1cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
困扰了我好久，查阅好多资料始终不得解，偶然之下将缩放比例调回**100%**，终于解决这个问题，这也是我目前遇到最大的坑，在此特别记录，以防自己以后忘记。

- 最后附上我学习时的参考资料：

[[Selenium2+python自动化45-18种定位方法](http://www.cnblogs.com/yoyoketang/p/6557421.html)](http://www.cnblogs.com/yoyoketang/p/6557421.html/)
[[Python爬虫利器五之Selenium的用法](http://www.cnblogs.com/BigFishFly/p/6380024.html)](http://www.cnblogs.com/BigFishFly/p/6380024.html)
