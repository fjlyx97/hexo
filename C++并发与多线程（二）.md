---
title: c++并发与多线程（二）
date: 2018-12-11 21:44:37
tags:
- 编程
- 学习
- C家族
- 多线程
categories : "C家族"
---

> - 承接上一篇C++并发与多线程（一）
> - 内容过多分篇记录
> - 主要涉及多个线程的创建
> - 互斥量的基本概念

<!--more-->

# 创建和等待多个线程
可以使用vector进行存储线程，实例代码如下：
```cpp
#include <iostream>
#include <thread>
#include <vector>

using namespace std;

void myprint(int inum)
{
    cout << "Id: " << std::this_thread::get_id() << endl;
    cout << inum << endl;
}

int main()
{
    vector<thread> mythreads;
    for (int i = 0 ; i < 10 ; i++)
    {
        mythreads.push_back(thread(myprint,i)); //这时候一旦创建就开始执行了
    }
    for (auto iter = mythreads.begin() ; iter != mythreads.end() ; iter++)
    {
        iter->join();
    }
    cout << "I love china" << endl;

    system("pause");
    return 0;
}
```

# 数据共享内存分析
## 只读数据
只读数据十分安全，不需要特别的处理手段，直接读取即可

## 有读有写数据
如果代码没有特殊处理，肯定崩溃。最简单的不崩溃处理：读的时候不能写，写的时候不能读

# 互斥量
保护共享数据，防止多个线程同时操作同一片内存空间，因此引入互斥量，将数据锁住
- lock以及unlock是mutex头文件中定义的函数，要成对使用，否则必然报错，示例代码如下：
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            my_mutex.lock();
            //cout << "Push data : " << i << endl;
            myData.push_back(i);
            my_mutex.unlock();
        }
        cout << "End pushData " << endl;
    }

    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                my_mutex.lock();
                //cout << "Pop data : " << myData.front() << endl;
                myData.pop_front();
                my_mutex.unlock();
            }
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
    
    return 0;
}
```

为了防止忘记unlock，引入类似智能指针的类模板std::lock_guard，可以同时取代lock以及unlock两个东西，因此不能再次lock以及unlock
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            std::lock_guard<std::mutex> sbguard(my_mutex);
            //cout << "Push data : " << i << endl;
            myData.push_back(i);
        }
        cout << "End pushData " << endl;
    }

    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                std::lock_guard<std::mutex> sbguard(my_mutex);
                //cout << "Pop data : " << myData.front() << endl;
                myData.pop_front();
            }
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
    return 0;
}
```
作用原理：std::lock_guard在语句块里面声明，构造函数上锁，出了语句块调用析构函数进行解锁，但是并不如手动上锁进行灵活

# 死锁
## 死锁的产生
死锁产生的前提条件是必须有两个锁（互斥量）才会产生，死锁演示:
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
    std::mutex my_mutex1;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            my_mutex.lock();
            my_mutex1.lock();
            cout << "Push data : " << i << endl;
            myData.push_back(i);
            my_mutex.unlock();
            my_mutex1.unlock();
        }
        cout << "End pushData " << endl;
    }

    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                my_mutex1.lock();
                my_mutex.lock();
                cout << "Pop data : " << myData.front() << endl;
                myData.pop_front();
                my_mutex.unlock();
                my_mutex1.unlock();
            }
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
    
    return 0;
}
```

## 死锁的一般解决方案
- 只要保证两个互斥量上锁顺序一致，就不会产生死锁

## std::lock()函数模板
功能：一次锁住两个或两个以上的互斥量（至少两个），不存在因锁头顺序问题而导致的死锁问题。如果其中一个互斥量没锁住，它就释放已经锁住的锁，直到所有互斥量都锁住。必须手动**unlock**
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
    std::mutex my_mutex1;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            std::lock(my_mutex,my_mutex1);
            cout << "Push data : " << i << endl;
            myData.push_back(i);
            my_mutex.unlock();
            my_mutex1.unlock();
        }
        cout << "End pushData " << endl;
    }
    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                std::lock(my_mutex,my_mutex1);
                cout << "Pop data : " << myData.front() << endl;
                myData.pop_front();
                my_mutex.unlock();
                my_mutex1.unlock();
            }
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
    
    return 0;
}
```

## 使用std::lock和std:lock_guard进行组合
默认情况下，std::lock_guard构造的变量会将变量上锁，但是可以手工指定第二个参数为std::adopt_lock进行忽略，std::adopt_lock为一个结构体对象，起到标记作用
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
    std::mutex my_mutex1;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            std::lock(my_mutex,my_mutex1);
            std::lock_guard<std::mutex> sblock(my_mutex,std::adopt_lock);
            std::lock_guard<std::mutex> sblock1(my_mutex1,std::adopt_lock);

            cout << "Push data : " << i << endl;
            myData.push_back(i);
        }
        cout << "End pushData " << endl;
    }
    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                std::lock(my_mutex,my_mutex1);
                std::lock_guard<std::mutex> sblock(my_mutex,std::adopt_lock);
                std::lock_guard<std::mutex> sblock1(my_mutex1,std::adopt_lock);
                cout << "Pop data : " << myData.front() << endl;
                myData.pop_front();
            }
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
    
    return 0;
}
```

# unique_lock取代lock_guard
unique_lock是个类模板，工作中一般lock_guard（推荐使用）。unique_lock比lock_guard灵活许多，但是灵活的代价是效率差了点，内存占用大了点。

在缺省的情况下，unique_lock和lock_guard第一个参数都相同，用起来区别不大。lock_guard第二个参数用std::adopt_lock作为标记，表示互斥量已经被标记，必须保证互斥量已经提前被lock过了，否则会报异常。同理unique_lock也可以用这个标记。

## std::try_to_lock
假设当下有两个线程在争夺一把锁，其中一个线程在拿到锁之后，线程进行休眠，这样子另一个线程则会因此而等待休眠结束。因此我们的第二个参数可以使用try_to_lock
- try_to_lock：尝试用mutex的lock去锁定这个mutex，但是如果没有锁定成功，会立即返回，并不会阻塞。但是自己千万不能进行手动**lock**，否则相当于lock两次

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
    std::mutex my_mutex1;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            //std::lock(my_mutex,my_mutex1);
            //std::lock_guard<std::mutex> sblock(my_mutex,std::adopt_lock);
            //std::lock_guard<std::mutex> sblock1(my_mutex1,std::adopt_lock);
            std::unique_lock<std::mutex> sblock(my_mutex,std::try_to_lock);
            if (sblock.owns_lock())
            {
                //cout << "Push data : " << i << endl;
                myData.push_back(i);
            }
            else
            {
                cout << "Thread1 cannot get lock" << endl;
            }
        }
        cout << "End pushData " << endl;
    }
    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                //std::lock(my_mutex,my_mutex1);
                //std::lock_guard<std::mutex> sblock(my_mutex,std::adopt_lock);
                //std::lock_guard<std::mutex> sblock1(my_mutex1,std::adopt_lock);
                std::unique_lock<std::mutex> sblock(my_mutex,std::try_to_lock);
                std::chrono::milliseconds dura(200000);
                std::this_thread::sleep_for(dura);
                if (sblock.owns_lock())
                {
                    cout << "Pop data : " << myData.front() << endl;
                    myData.pop_front();
                }
                else
                {
                    cout << "Cannot get lock" << endl;
                }
            }
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
    
    return 0;
}
```

## std::defer_lock
用这个defer_lock的前提是，你不能自己先lock，否则会报异常。defer_lock就是初始化一个没有加锁的mutex

## unique_lock的成员函数
1. lock() 加锁
2. unlock() 解锁
3. try_lock() 尝试给互斥量加锁，拿到锁返回true，这个函数不阻塞
4. release() 返回它所管理的mutex对象指针，并释放所有权。也就是说这个unique_lock和mutex不再有关系（绑定在一起的关系进行分开），如果原来mutex处于加锁状态，有责任接管过来自己解锁。
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
    std::mutex my_mutex1;
public:
    A() { cout << "construct A " << endl;}
    A(const A* mA) { cout << "copy construct A " << endl;}

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            cout << "Push data : " << i << endl;
            std::unique_lock<std::mutex> sbguard(my_mutex);
            std::mutex* ptx = sbguard.release();
            ptx->unlock();
            myData.push_back(i);
        }
        cout << "End pushData " << endl;
    }
    void popData()
    {
        cout << "Enter popData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
            if(!myData.empty())
            {
                cout << "Pop data : " << myData.front() << endl;
                std::unique_lock<std::mutex> sbguard1(my_mutex1,std::defer_lock);
                sbguard1.lock();
                myData.pop_front();
            }
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
    
    return 0;
}
```

## unique_lock所有权的传递
unique_lock和mutex进行绑定到一起从而发挥作用，假设std::unique_lock<std::mutex> sbguard1(my_mutex1)，sbguard1可以把my_mutex1所有权转移给unique_lock其他对象，但是不允许复制，和智能指针一样

```cpp
std::unique_lock<std::mutex> sbguard2(std::move(my_mutex1)); //移动语义，相当于sbguared2和my_mutex1绑定到了一起

//另一种写法
class A {
public:
    std::unique_lock<std::mutex> rtn_unique_lock()
    {
        std::unique_lock<std::mutex> tmpguard(my_mutex1);
        //会自动生成unique_lock对象，并调用移动构造函数
        return tempguard;
    }
};
```