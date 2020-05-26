---
title: QT自学之路
date: 2018-01-01 19:40:20
tags:
- 编程
- 学习
- 图形
- QT
categories : "QT学习记录"
---

>- QT作为一个图形库，可以以简单的代码构造出图形界面
>- 在此记录QT的学习记录

<!--more-->

# QDialog
- 对话框有模态与非模态对话框
- 模态对话框窗口置顶，必须先点击它
```cpp
//模态对话框
QDialog dialog(this);
dialog.exec();
//非模态对话框
QDialog * dialog = new QDialog(this);
dialog->show();
//dialog->setModal(ture);这种写法也可创建模态对话框
```
- QDialog如果按下后，QDialog.exec() == QDialog::Accepted

## 颜色对话框
- 包含在 QColorDialog头文件当中
```cpp
QColor color = QColorDialog::getColor(Qt::red,this,"颜色对话框");
```

## 文件对话框
- 包含在 QFileDialog头文件当中
```cpp
//返回文件路径
QString filename = QFileDialog::getOpenFileName(this,"文件对话框","D","文件图片(*.jpg *.png");
qDebug() << "filename is :" << filename << endl;
```

## 字体对话框
- 包含在 QFontDialog头文件当中
```cpp
bool ok;
QFont font = QFontDialog::getCont(&ok,this);
if (ok)
{
    ui->pushButton->setFont(font);
    else
        qDebug() << "没有选择字体";
}

## 输入对话框
- 包含在头文件 QInputDialog当中
```cpp
bool ok;
QString string = QInputDialog::getText(this,"输入字符串对话框","请输入用户名",Q)LineEdit::Normal,"admin",&ok);
```

## 消息对话框
- 包含QMessageBox头文件中
```cpp
int ret1 = QMessageBox::question(this,"问题对话框","你了解qt吗？",QMessageBox::Yes,QMessageBox::No);
```

## 进度对话框
- 包含在 QProgressDialog当中
- 一般是模态对话框，如果要非模态对话框，需要借助定时器的帮助
```cpp
//QProgressDialog* dialog = new QProgressDialog("文件对话框","取消",0,100,this);
QProgressDialog dialog("文件复制进度","取消",0,50000,this);
//dialog->show();
dialog.show();
for ( int i = 0 ; i < 500000 ; i++)
{
    dialog.setValue(i);
    QCoreApplication::processEvents();
    if (dialog.wasCanceled())
    {
        break;
    }
}
dialog.setValue(50000)
```

# 信号与槽
- 槽一般以**public slot**修饰，写在头文件当中，在源文件中实现定义
- 可以使用alt+enter来快速添加定义
- 使用connect来连接，具体代码如下
```cpp
//button是一个指针变量
//第二个参数是按钮自带的选项
//第三个参数代表按钮与当前窗口相连接
//第四个参数代表已经定义的槽函数
//复杂点说依次为：发射信号的对象，发射的信号，接受信号的对象，要执行的槽函数
connect(button,&QPushButton::clicked,this,&MainWindow::showChildDialog);
```

# QLabel
看文档