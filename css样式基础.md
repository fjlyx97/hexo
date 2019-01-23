---
title: css样式基础
date: 2019-01-17 14:17:34
tags:
- css
- 网页
categories: 网页设计
---

> - 网页设计一直不大美观
> - 尝试通过视频课程学习CSS
> - 涉及CSS基础样式，选择器
> - 常用标签归类
> - 盒子模型
> - 定位技术

<!-- more -->

# 引入CSS
## 注释
```
/* */
```
## 行间样式
- 使用style属性进行引入
```html
<div style="
    width:100px;
    height:100px;"
</div>
```

## 页面级CSS 
- 使用style标签
```html
<style type="text/css">
    div {
        height: 100px;
    }
</style>
```

## 引入外部CSS
- 使用link标签
```html
<link rel="stylesheet" href="">
```

# 选择器

## id选择器
在头标签使用标签取名
- html
```html
<div id="only">123</div>
```
- css
```css
#only {
    background-color: green;
}
```
一个元素只能有**一个ID**值，一一对应

## class选择器
与id选择器不同的是，一个class可以对应多个元素
- html
```html
<div class="demo">cssdemo</div>
```
- css
```css
.demo{
    background-color: green;
}
```

## 标签选择器
所有的标签都可以选择，如下：
- css
```css
div{
    background-color: green;
}
```

## 通配符选择器
全选所有的标签，如下：
- css
```css
* {
    font-size: 100px;
}
```

## 属性选择器
如果拥有这个属性，则可以全部选出来
- css
```css
[id] {
    font-size: 100px;
}
[id="only"] {
    font-size: 100px;
}
```

## 优先级
### 选择器
!important > 行间样式 > id > class || 属性选择器 > 标签选择器 > 通配符

- class 和 属性选择器，按先后顺序来确定

# 复杂选择器
## css权重
```
1. important             Infinity
2. 行间样式               1000
3. id                     100
4. class|属性|伪类         10
5. 标签|伪元素              1
6. 通配符                   0
```

数值为256进制，举例行间样式如下:
```
1     0     0     0
0-255 0-255 0-255 0-255
```
- 如果所有选择器写在了同一行，则将他们的权重相加，进行判定

## 父子选择器/派生选择器
- html
```html
<div id="only">
    <span>hello</span>
</div>
```
- css
```css
div span{
    font-size: 20px;
}
```
- 既然是父子选择器，则父亲下所有的子类都可以被覆盖
- 不一定非要使用标签选择器，同样可以用其他选择来替代
- 浏览器底层为自右向左遍历

## 直接子元素选择器
不同于父子选择器，这代表父级下直接的子元素
- html
```html
<div id="only">
    <span>hello</span>
</div>
```
- css
```css
div > span{
    font-size: 20px;
}
```

## 并列选择器
中间不允许**加空格**，且顺序不能引起歧义
- html
```html
<div>1</div>
<div class="demo">2</div>
<p class="demo">3</p>
```
- css
```css
div.demo{
    font-size: 20px;
}
```

## 分组选择器
将所有东西分为一组，以逗号分隔
- css
```css
em,
strong,
span{
    font-size: 30px;
}
```

# css基本属性
- border:1px solid black; 边框粗细 属性 颜色，对应border-width border-style border-color，同样可是用border-left或其他指定位置设置

## 文本属性
```
text-align: center; 文字对齐方式
text-indent: 2em;  文字缩进，em为文本距离，1em=1*font-size
text-decoration: line-through; 文本装饰，等效<del>
line-height: 16px; 单行文本高度，类比word的行距
cursor: pointer; 设置鼠标指到的光标样式
```
- 设置单行文本垂直居中，通用手法为文本高度等于容器高度，即line-height == height，还有其他方法为：padding

## 字体属性
```
font-size : 30px; 字体大小，设置字体高度
font-weight : bold; 设置字体属性，如加粗
font-style : italic; 设置字体样式
font-family: arial; 设置字体，如微软雅黑
color: red; 设置字体颜色
```
- 字体颜色有三种格式，纯英文单词，颜色代码，颜色函数。颜色代码如：#ff44ff，每两位代表一个颜色，分别代表rgb,范围从00-ff，或使用rgb(0-255,0-255,0-255)

## 伪类选择器
先选中标签，加上冒号，如下为伪类选择器之一
```css
a:hover {
    background-color: orange;
}
```

# 标签归类
拥有隐式属性display
## 行级元素 inline
### 特性
- 内容决定元素所占位置
- 不可以通过CSS改变宽高
### 常用行级元素
1. span
2. strong
3. em
4. a
5. del

## 块级元素 block
### 特性
- 独占一行
- 可以通过CSS改变宽高
### 常用块级元素
1. div
2. p
3. ul
4. li
5. ol
6. form
7. address

## 行级块元素 inline-block
### 特性
- 内容决定大小
- 可以改变宽高
### 常用行级块元素
1. img ，在修改宽高时，可以只设置其中一个，会自动等比例缩放，如果两个都修改的话有可能会出现比例失衡
- 凡是带有inline的元素，都有文字特性，如下例子：
```html
<img src="1.jpg">
<img src="1.jpg">
<img src="1.jpg">
<img src="1.jpg">
<!-- 四张图片出现缝隙 -->
```

# 盒子模型
盒子模型分为四大部分
- 外边距 margin
```
body拥有默认的margin，为8像素，所以常使用通配符选择器进行初始化
```
- 盒子壁 border
- 内边距 padding
```
padding: 100px 100px 100px 100px
        上      右     下    左
padding: 100px 100px 100px
            上  左右  下
padding: 100px 100px 
            上下 左右
padding: 100px
四个方向

其余同理
```
- 盒子内容 width+height

# 定位技术
样式中拥有一个参数为position，且拥有left,right,top,bottom可以与之相配合，使用z-index设置位于层数
- left和right不能同时出现，分别为左边线距离和边线距离

## 绝对定位
使用absolute，进行绝对定位。会脱离原来的层，跑到上一层，因此下一个元素会覆盖之前的位置
- 相对于最近的有定位的父级进行定位，如果寻找不到，相对于Document进行定位

## 相对定位
使用relative，进行相对定位。保留原来位置进行定位，即下一个位置无法覆盖之前的位置。
- 相对于自己的出生位置进行定位

## 固定定位
使用fixed，进行固定定位，不管怎么移动始终在同个位置

## 居中样例
如下css:
```css
div{
    position: absolute;
    left: 50%;
    height: 50%;
}
```

- relative常用于支架而absolute常用于移动元素

# 两个经典BUG
## 第一个BUG（margin塌陷）
当子集元素使用margin-top时，无法下移，只有大于父亲的marginn-top时，才能将整体下移
- 垂直方向的margin，会粘合到一起，取最大的值
### 解决方案
#### 第一个解决方案
可以通过在父级元素添加border-top:1px solid red;进行修复（不推荐）
#### 第二个解决方案
触发一个盒子的BFC
- position:absolute;
- display:inline-block;
- float:left/right;
- overflow:hidden; //溢出部分隐藏

## 第二个BUG（margin合并）
当两个元素，一个元素使用margin-bottom:100px，另外一个元素使用margin-top:200px，他们的真实距离不是300，而是取最大值，但是这种BUG通常不会使用BFC进行修复，我们默认他们是存在的，而是通过计算像素的方法进行手动修复

# 浮动模型
float 元素仅仅具有两个选项：left以及right，作用为元素站队，中间同样可以加其他元素
- html如下
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>first html</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" href="./index.css">
</head>
<body>
    <div class="wrapper">
        <div class="content">1</div>
        <div class="content">2</div>
        <div class="content">3</div>
        <div class="content">4</div>
        <div class="content">5</div>
        <div class="content">6</div>
        <div class="content">7</div>
        <div class="content">8</div>
        <div class="content">9</div>
    </div>
</body>
<
```
- css如下
```css
* {
    margin: 0;
    border: 0;
}

.wrapper{
    width: 300px;
    height: 300px;
    border: 1px solid black;
}

.content{
    float: left;
    width: 100px;
    height: 100px;
    background-color: green;
}
```

## 原理
浮动元素产生了浮动流，所有产生了浮动流的元素，块级元素看不到他们，产生了BFC的元素和文本类属性的元素以及文本都能看到浮动元素

## 清除浮动流
使用 clear:both即可以清除浮动流

## 伪元素
一个元素出生的时候就具有伪元素，如下代码：
- html
```html
<span>
    very good!
</span>
```
- css
```css
span::before{
    Hi
}
span::after{
    content:"";
    display:block;
    clear:both;
}
```

伪元素可以被CSS正常操作，但是不能作为结构。且伪元素天生是行级元素，可以通过修改display进行修改。因此可以通过在伪元素添加clear让最后的元素可以看到浮动元素，因此块级元素可以自适应

- 凡是涉及了positon:abolute，float:left/right，打内部把元素转换成inline-block

# 文字溢出处理
## 单行文本
1. 先让文本失去换行功能，white-space:nowrap;
2. 设置溢出部分隐藏，overflow:hidden;
3. 设置溢出部分用点号，text-overflow: ellipsis;

## 多行文本
由于浏览器版本不齐，因此常将文字直接传入后台，进行截断处理，手动添加...

# 背景图片处理
使用div时，允许添加背景图片
- css
```css
.logo {
    width: 100px;
    height: 100px;
    background-image: url('./1.jpg');
    background-repeat: no-repeat;
}
```

## 图片代替文字
## 第一种解决方案
当网速低的时候，浏览器使用默认加载策略，禁用JavaScript以及CSS，这样如果没做特殊处理，图片挂掉后就不能点击链接。因此我们可以使用以下代码来隐藏链接文字：
- text-indent:容器宽度，
- white-space: nowrap;
- overflow: hidden;
## 第二种解决方案
容器高度设置为0，padding-top设置为图片高度，使用overflow:hidden来隐藏文字

# 规定
- p标签不能嵌套div元素
- a标签不能嵌套a标签