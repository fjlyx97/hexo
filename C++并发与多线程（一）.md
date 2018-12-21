---
title: c++并发与多线程（一）
date: 2018-12-09 20:47:37
tags:
- 编程
- 学习
- C家族
- 多线程
categories : "C家族"
---

> - C++小白进阶之路
> - 了解多线程是如何编写的
> - 跟随网易云课堂视频学习
> - 内容涵盖线程启动，创建，传参以及detach的坑点

<!--more-->

# 并发，进程，线程
两个或者更多的任务同时发生进行，一个程序同时执行多个任务，由操作系统统一调度

进程与主线程唇齿相依，线程可以近似理解成执行代码的通路，且主线程唯一

线程并不是越多越好，每个线程都需要一个独立的堆栈空间（1M理论值），线程需要保存中间状态，耗费本该属于程序运行的时间

## 多进程并发
进程之间的通信（同一个电脑：管道，文件，消息队列，共享内存）（不同电脑：socket通信技术）

## 多线程并发
每个线程都有自己独立的路径，但是一个进程所有线程共享地址空间（共享内存），全局变量，指针，引用都可以在线程之间传递，因此使用多线程的开销远远小于多进程

共享内存带来新问题，数据是否为一致性（线程锁等）

多进程并发和多线程并发虽然可以混合使用，但是建议优先考虑多线程技术而不是多进程

## C++11新标准线程库
以往Windows：CreateThread() , _beginthread(),_beginthreadexe()

以往Linux:pthread_create()

以往多线程不能跨平台，C++11新标准允许。

# 线程创建，启动，结束
进程结束的标志是主线程是否执行完毕，如果主线程执行完毕了，代表整个进程执行完毕。（一般情况）如果其他子线程还没有执行完毕，那么这些子线程也会被操作系统强行终止。因此得出结论：如果想保持子线程的运行状态，需要保持主线程一直运行。（但是有例外情况）

## 范例（主线程等待子线程）
```cpp
#include <iostream>

//包含thread头文件
#include <thread>
//创建一个函数
void myprint()
{
    std::cout << "Thread begin.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    return;
}

int main()
{
    std::thread myobj(myprint);
    //阻塞主线程，并等待myprint子线程执行完毕
    myobj.join();
    std::cout << "I love China!" << std::endl;
    system("pause");
    return 0;
}

```
- join阻塞主线程，当子线程执行完毕，才可以继续往下运行

## 范例（主线程不等待子线程）
传统多线程程序，主线程需要等待子线程执行完毕，然后自己再最后退出

detach分离，一旦detach之后，与主线程关联的thread对象，就会失去与主线程的关联，此时子线程就会驻留在后台运行，（主线程跟子线程相当于被C++运行时库接管），当子线程执行完毕之后，由运行时库负责清理该线程相关的资源（守护线程）

```cpp
#include <iostream>

//包含thread头文件
#include <thread>
//创建一个函数
void myprint()
{
    std::cout << "Thread begin.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    return;
}

int main()
{
    std::thread myobj(myprint);
    myobj.detach();
    std::cout << "I love China!" << std::endl;

    system("pause");
    return 0;
}
```

## joinable函数
判断是否可以成功调用join或者detach函数，返回true或者false

```cpp
#include <iostream>

//包含thread头文件
#include <thread>
//创建一个函数
void myprint()
{
    std::cout << "Thread begin.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    std::cout << "Thread end.." << std::endl;
    return;
}

int main()
{
    std::thread myobj(myprint);
    myobj.join();
    if (myobj.joinable())
    {
        std::cout << "1:joinable() == true" << std::endl;
    }
    else
    {
        std::cout << "2:joinable() == false" << std::endl;
    }
    std::cout << "I love China!" << std::endl;

    system("pause");
    return 0;
}
```

一旦join或者detach之后，joinable返回false，再次调用引发异常

# 其他创建线程的手法
## 用类进行调用
```cpp
#include <iostream>
#include <thread>
using namespace std;
class TA
{
public:
    void operator()()
    {
        cout << "我的线程开始执行了" << endl;
        cout << "我的线程结束了" << endl;
    }
};

int main()
{
    TA ta;
    thread myobj(ta);
    myobj.join();
    cout << "I love China!" << endl;

    system("pause");
    return 0;
}
```

## 使用detach的坑
```cpp
#include <iostream>
#include <thread>
using namespace std;
class TA
{
public:
    int &m_i;
    TA(int &i) : m_i(i)
    {

    }
    void operator()()
    {
        cout << "m_i1 value is " << m_i << endl;
        cout << "m_i2 value is " << m_i << endl;
        cout << "m_i3 value is " << m_i << endl;
        cout << "m_i4 value is " << m_i << endl;
        cout << "m_i5 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
        cout << "m_i6 value is " << m_i << endl;
    }
};

int main()
{
    int myi = 6;
    TA ta(myi);
    thread myobj(ta);
    myobj.detach();
    cout << "I love China!" << endl;

    return 0;
}
```

由于传入的myi是引用，当程序结束时，变量被回收，子线程将是不确定的值

新问题出现了，当程序结束的时候，对象被回收了，但是却不影响调用，因此得出结论，对象是被复制到线程当中去的，ta被销毁，所复制的ta对象依然存在

## 用lambda表达式
```cpp
#include <iostream>
#include <thread>
using namespace std;
int main()
{
    auto mylambda = []{
        cout << "线程开始执行了" << endl;
    };
    thread myobj(mylambda);
    myobj.join();
    cout << "I love China!" << endl;

    system("pause");
    return 0;
}
```

# 线程传参
## 传递临时对象作为线程参数

```cpp
#include <iostream>
#include <thread>
using namespace std;
void myprint(const int &i,char* pmybuf)
{
    //不推荐用引用传参，绝对不要用指针
    cout << i << endl; //分析认为,i的地址和main函数中i地址不相同，因此i的值是安全的（i被复制了一份）
    cout << pmybuf << endl; //pmybuf指向的是main函数中的mybuf，不安全
    return;
}
int main()
{
    int mvar = 1;
    int& mvary = mvar;
    char mybuf[] = "this is a test!";
    thread myobj(myprint,mvar,mybuf);
    myobj.detach();
    cout << "I Love China" << endl;
    return 0;
}
```

- （陷阱1）如上可以看出参数为引用虽然可以，但是不大推荐，指针则是极其危险的。但是一定要传的话，可以尝试将char\* pmybuf 更改为const string& pmybuf（依然有BUG）

- （陷阱2）mybuf到底在何时转为string的？因此可以用如下改法

```cpp
#include <iostream>
#include <thread>
#include <string>
using namespace std;
void myprint(const int &i,const string& pmybuf)
{
    //不推荐用引用传参，绝对不要用指针
    cout << i << endl; //分析认为,i的地址和main函数中i地址不相同，因此i的值是安全的（复制的）
    cout << pmybuf << endl; //pmybuf指向的是main函数中的mybuf，不安全
    return;
}
int main()
{
    int mvar = 1;
    int& mvary = mvar;
    char mybuf[] = "this is a test!";
    //thread myobj(myprint,mvar,mybuf); //存在mybuf被回收了，才开始复制
    //正确写法
    thread myobj(myprint,mvar,string(mybuf)); 
    myobj.detach();
    cout << "I Love China" << endl;

    return 0;
}
```

## 小结
- 若传递int这种简单类型参数，建议都是值传递，不要用引用，防止节外生枝
- 如果传递类对象，避免隐式类型转换，全部都在创建线程时，构造出临时对象，函数体使用引用来接
- 终极结论：建议不使用detach，只使用join，这样就不存在局部变量失效导致对内存的非法引用。

## 传递临时对象作为线程参数（续）
线程id可以通过标准库函数获得如下：
```cpp
std::this_thread::get_id();
```

## 传递类对象、智能指针作为线程参数
### 类对象
默认往线程中传递类对象时，假设目前有类A，即使自己使用的是const A& a，依然会拷贝构造出一个类对象，且const必须加，不加新型编译器会报错。

std::ref函数则可以真正将一个对象传递进去，且const可以去掉，成员变量也不需要使用mutable关键字，ref函数传入的类对象不会被复制，因此主线程和新建的线程使用的是同一个对象，写法如下：

```cpp
class A;
std::ref(A);
```

### 智能指针
unique_ptr智能指针直接作为参数传入会报错，因此需要使用std::move进行传输，代码如下：

```cpp
#include <iostream>
#include <thread>
#include <string>
using namespace std;
void myprint(unique_ptr<int> i)
{
    //不推荐用引用传参，绝对不要用指针
    cout << *i << endl; //分析认为,i的地址和main函数中i地址不相同，因此i的值是安全的（复制的）
    cout << "Thread new : " << std::this_thread::get_id() << endl;
    return;
}
int main()
{
    unique_ptr<int> myp(new int(100));
    thread myobj(myprint,std::move(myp)); 
    myobj.join();

    system("pause");
    return 0;
}
```

### 用成员函数指针作为线程函数
第一个参数为成员函数的地址，第二个参数为传入的类成员

```cpp
#include <iostream>
#include <thread>
#include <string>
using namespace std;
class A
{
public:
    void say()
    {
        cout << "Son thread begin.." << endl;
    }
};
int main()
{
    A myobj;
    thread mytobj(&A::say,&myobj);
    mytobj.join();

    system("pause");
    return 0;
}
```
类成员尽量使用引用传入函数，可以保证使用的是同一个类成员而不被复制一份