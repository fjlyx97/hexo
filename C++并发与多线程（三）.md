---
title: c++并发与多线程（三）
date: 2018-12-14 14:34:37
tags:
- 编程
- 学习
- C家族
- 多线程
categories : "C家族"
---

> - 承接上一篇C++并发与多线程（二）
> - 内容过多分篇记录
> - 设计单例模式共享数据分析
> - std::call_once的使用
> - 进程间的通讯

<!--more-->

# 设计模式概括
用设计模式写出来的代码程序灵活，维护起来可能方便，但是写出来的代码很晦涩，别人接管代码，阅读代码不大方便

## 单例设计模式
单例设计模式使用频率较高，单例：整个项目中，由某个或者或某些特殊的类，属于该类的对象，只允许创建一个

### 单例类
```cpp
#include <iostream>
using namespace std;
class MyCAS
{
private:
    MyCAS()  //私有化了构造函数
    {

    } 
private:
    static MyCAS *m_instance; //静态成员变量

public:
    static MyCAS* GetInstance()
    {
        if (m_instance == NULL)
        {
            m_instance = new MyCAS();
        }
        static CGarhuishou cl; //静态对象生命周期一直到程序退出
        return m_instance;
    }

    class CGarhuishou //类中套类，用来释放对象
    {
        public:
            ~CGarhuishou()
            {
                if (MyCAS::m_instance)
                {
                    delete MyCAS::m_instance;
                    MyCAS::m_instance = NULL;
                }
            }
    };

    void func()
    {
        cout << "Test " << endl;
    }
};
MyCAS* MyCAS::m_instance = NULL;

int main()
{
    MyCAS* p_a = MyCAS::GetInstance(); //创建一个对象，返回该类的指针

    system("pause");
    return 0;
}
```
单例模式本质就是将构造函数私有化，设置静态成员函数进行初始化成员，一旦类中的静态成员不为NULL，就New一个出来，否则返回静态成员

- 值得一提的是要如何释放它已经new出来的空间，在这里我们要在类中再套一个类，并在构造成员的同时，声明一个static的对象，这个对象会在程序整个生命周期结束的时候，调用自身的析构函数，从而释放整片空间

### 单例设计模式共享数据分析、解决
为了共享单例类的数据，我们引入多线程，但是原先的代码有可能碰到多次new同一个类成员，因此我们需要引入锁，进行双重保护，代码如下：
```cpp
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;
std::mutex rescource_mutex;
class MyCAS
{
private:
    MyCAS()  //私有化了构造函数
    {

    } 
private:
    static MyCAS *m_instance; //静态成员变量

public:
    static MyCAS* GetInstance()
    {
        // 提高效率
        if (m_instance == NULL) //双重锁定，提高效率
        {
            std::unique_lock<std::mutex> mymutex(rescource_mutex);
            if (m_instance == NULL)
            {
                m_instance = new MyCAS();
            }
            static CGarhuishou cl; //静态对象生命周期一直到程序退出
        }
        return m_instance;
    }

    class CGarhuishou //类中套类，用来释放对象
    {
        public:
            ~CGarhuishou()
            {
                if (MyCAS::m_instance)
                {
                    delete MyCAS::m_instance;
                    MyCAS::m_instance = NULL;
                }
            }
    };

    void func()
    {
        cout << "Test " << endl;
    }
};
MyCAS* MyCAS::m_instance = NULL;

void initThread()
{
    cout << "In" << endl;
    MyCAS::GetInstance();
    cout << "Out" << endl;
    return;
}

int main()
{
    std::thread t1(initThread);
    std::thread t2(initThread);

    t1.join();
    t2.join();
    system("pause");
    return 0;
}
```

### std::call_once() 函数模板
C++11引入的函数，其中第二个参数为函数名，功能：能够保证多线程的情况下，函数A只被调用一次
- 具备互斥量的能力
- 据说比互斥量消耗的资源更少
- 需要结合标记结合使用，std::once_flag，是一个结构，只要once_flag被设置为已调用的状态，后续的函数就不会再被执行了

```cpp
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;
std::mutex rescource_mutex;
std::once_flag g_flag;

class MyCAS
{
    static void createInstance()
    {
        m_instance = new MyCAS();
        static CGarhuishou cl; //静态对象生命周期一直到程序退出
    }
private:
    MyCAS()  //私有化了构造函数
    {
    } 
private:
    static MyCAS *m_instance; //静态成员变量

public:
    static MyCAS* GetInstance()
    {
        std::call_once(g_flag,createInstance);
        cout << "CallOnce over" << endl;
        return m_instance;
    }

    class CGarhuishou //类中套类，用来释放对象
    {
        public:
            ~CGarhuishou()
            {
                if (MyCAS::m_instance)
                {
                    delete MyCAS::m_instance;
                    MyCAS::m_instance = NULL;
                }
            }
    };
    void func()
    {
        cout << "Test " << endl;
    }
};
MyCAS* MyCAS::m_instance = NULL;

void initThread()
{
    cout << "In" << endl;
    MyCAS::GetInstance();
    cout << "Out" << endl;
    return;
}

int main()
{
    std::thread t1(initThread);
    std::thread t2(initThread);

    t1.join();
    t2.join();
    system("pause");
    return 0;
}
```

- 单例模式建议在主线程先创建完对象之后，再开多线程

# 进程间的通讯
## std::condition_variable类
本质上为一个类，等待条件达成，需要和互斥量进行配合工作
- 需要引入头文件condition_variable
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>
#include <condition_variable>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex1;
    std::condition_variable my_cond;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 1000 ; i++)
        {
            std::unique_lock<std::mutex> sbguard(my_mutex1);
            cout << "Push data : " << i << endl;
            myData.push_back(i);
            my_cond.notify_one();  //尝试唤醒wait的线程
        }
        cout << "End pushData " << endl;
    }
    void popData()
    {
        cout << "Enter popData " << endl;
        while(true)
        {
            std::unique_lock<std::mutex> sbguard1(my_mutex1);
            //wait用来等一个东西
            //如果第二个参数（lambda）返回值为false,wait解锁互斥量，并堵塞到本行
            //如果返回值为true,wait()直接返回
            //堵塞到其他某个线程调用notify_one()成员函数为止
            //如果缺少第二个参数，结果就和lambda返回false一样
            //一旦被唤醒，会重新进行判断，如果返回true就继续往下执行
            my_cond.wait(sbguard1,[this]{
                if (!myData.empty())
                    return true;
                return false;
            });
            myData.pop_front();
            sbguard1.unlock();
            cout << "Pop Data" << endl;
        }
        cout << "End popData " << endl;
    }
};

int main()
{
    A tempA;
    std::thread thread1(&A::pushData,&tempA);
    std::thread thread2(&A::popData,&tempA);
    thread1.join();
    thread2.join();
    
    system("pause");
    return 0;
}
```
- wait代表用来等一个东西
- 如果第二个参数（lambda）返回值为false,wait解锁互斥量，并堵塞到本行
- 如果返回值为true,wait()直接返回
- 堵塞到其他某个线程调用notify_one()成员函数为止
- 如果缺少第二个参数，结果就和lambda返回false一样
- 一旦被唤醒，会重新进行判断，如果返回true就继续往下执行

## notify_all()
某些情况下，我们可以同时唤醒多个线程，因此代码更改如下：
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>
#include <condition_variable>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex1;
    std::condition_variable my_cond;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 1000 ; i++)
        {
            std::unique_lock<std::mutex> sbguard(my_mutex1);
            cout << "Push data : " << i << endl;
            myData.push_back(i);
            my_cond.notify_all();  //尝试唤醒wait的线程
        }
        cout << "End pushData " << endl;
    }
    void popData()
    {
        cout << "Enter popData " << endl;
        while(true)
        {
            std::unique_lock<std::mutex> sbguard1(my_mutex1);
            //wait用来等一个东西
            //如果第二个参数（lambda）返回值为false,wait解锁互斥量，并堵塞到本行
            //如果返回值为true,wait()直接返回
            //堵塞到其他某个线程调用notify_one()成员函数为止
            //如果缺少第二个参数，结果就和lambda返回false一样
            //一旦被唤醒，会重新进行判断，如果返回true就继续往下执行
            my_cond.wait(sbguard1,[this]{
                if (!myData.empty())
                    return true;
                return false;
            });
            myData.pop_front();
            cout << "Thread id : " << std::thread::get_id << endl;
            cout << "Pop Data" << endl;
            sbguard1.unlock();
        }
        cout << "End popData " << endl;
    }
};

int main()
{
    A tempA;
    std::thread thread1(&A::pushData,&tempA);
    std::thread thread2(&A::popData,&tempA);
    std::thread thread3(&A::popData,&tempA);
    thread1.join();
    thread2.join();
    thread3.join();
    
    system("pause");
    return 0;
}
```