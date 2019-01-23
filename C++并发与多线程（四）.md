---
title: c++并发与多线程（四）
date: 2018-12-22 16:45:37
tags:
- 编程
- 学习
- C家族
- 多线程
categories : "C家族"
---

> - 承接上一篇C++并发与多线程（三）
> - 内容过多分篇记录
> - 涉及async,future创建后台应用
> - std::packaged_task
> - std::promise
> - std::future其他函数
> - 原子操作
> - 自动析构技术
> - 补充知识点

<!--more-->

# async,future创建后台应用
如果希望线程返回结果,我们可以依赖async或者feture，需要引入future头文件

## async
std:async 是个函数模板，用来启动一个异步任务，它返回一个std::feture对象
- 启动异步任务：自动创建一个线程并执行对应的线程入口函数
### 使用普通函数
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
int mythread() //线程入口函数
{
    cout << "mythread start " << "Thread id::" << std::this_thread::get_id() << endl;
    std::chrono::millisecond dura(50000); //休息五秒
    std::this_thread::sleep_for(dura);
    cout << "mythread end" << endl;
    return 5;
}
int main()
{
    cout << "main : " << std::this_thread::get_id() << endl;
    //代表async的对象和future绑定在一起
    std::future<int> result = std::async(mythread);
    cout << "continue : " << result.get() << endl;
    //还有一个wait函数，等待线程返回，本身不返回结果
    //result.wait();

    system("pause");
    return 0;
}
```
- 使用get类成员函数来获得返回值
- wait函数只等待线程，并不返回结果

### 使用类成员函数
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
class A
{
public:
    int mythread(int mypar) //线程入口函数
    {
        cout << "mypar : " << mypar << endl;
        cout << "mythread start " << "Thread id::" << std::this_thread::get_id() << endl;
        std::chrono::millisecond dura(50000); //休息五秒
        std::this_thread::sleep_for(dura);
        cout << "mythread end" << endl;
        return 5;
    }
};
int main()
{
    A a;
    int tmppar = 12;
    cout << "main : " << std::this_thread::get_id() << endl;
    std::future<int> result = std::async(&A::mythread,&a,tmppar);
    cout << result.get() << endl;

    system("pause");
    return 0;
}
```

### async额外参数
我们可以通过向std::async()传递一个参数，类型为std::launch类型（枚举类型），来达到一些特殊的目的
- std::launch::deferred:表示线程入口调用被延迟到std::future的wait()或get()函数被调用时才执行，延迟调用，并且没有创建新线程，在主线程中调用的线程入口函数
- std::launch::async:在调用async的时候就创建了线程，且强制创建线程
- 若不填写，默认为 std::launch::async | std::launch::deferred
测试代码如下：
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
class A
{
public:
    int mythread(int mypar) //线程入口函数
    {
        cout << "mypar : " << mypar << endl;
        cout << "mythread start " << "Thread id::" << std::this_thread::get_id() << endl;
        std::chrono::millisecond dura(50000); //休息五秒
        std::this_thread::sleep_for(dura);
        cout << "mythread end" << endl;
        return 5;
    }
};
int main()
{
    A a;
    int tmppar = 12;
    cout << "main : " << std::this_thread::get_id() << endl;
    std::future<int> result = std::async(std::launch::async,&A::mythread,&a,tmppar);
    cout << result.get() << endl;

    system("pause");
    return 0;
}
```

### std::async深入谈
如果系统资源紧张，很有可能在创建thread时会失败，由于第一个参数默认为 std::launch::async | std::launch::deferred，由系统来确定同步执行还是异步执行（是否创建新线程）

## std::packaged_task
std::packaged_task为类模板，模板参数为各种可调用对象，通过这个类模板，可以包装各种可调用对象，方便将来作为线程入口函数来调用
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
int mythread(int mypar) //线程入口函数
{
    cout << "mypar : " << mypar << endl;
    cout << "mythread start " << "Thread id::" << std::this_thread::get_id() << endl;
    std::chrono::millisecond dura(50000); //休息五秒
    std::this_thread::sleep_for(dura);
    cout << "mythread end" << endl;
    return 5;
}
int main()
{
    std::packaged_task<int(int)> mypt(mythread);//返回值为int（尖括号），参数也为int（圆括号）
    std::thread t1(std::ref(mypt),1);
    std::future<int> result = mypt.get_future(); //使用get_future，然后就可以使用get获得返回值了
    cout << result.get() << endl;
    t1.join();

    system("pause");
    return 0;
}
```
写法如上，可以使用get_future函数，来返回std::futrue对象，因此还可以继续使用get函数进行获得返回值

### 包装lambda表达式
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
int main()
{
    std::packaged_task<int(int)> mypt([](int i){
        cout << "i is : " << i << endl;
        return i;
    });//返回值为int（尖括号），参数也为int（圆括号）
    std::thread t1(std::ref(mypt),1);
    std::future<int> result = mypt.get_future(); //使用get_future，然后就可以使用get获得返回值了
    cout << result.get() << endl;
    t1.join();

    system("pause");
    return 0;
}
```

package_task本身就是可以调用的对象，因此可以不用创建线程，代码如下：
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
int main()
{
    std::packaged_task<int(int)> mypt([](int i){
        cout << "i is : " << i << endl;
        return i;
    });//返回值为int（尖括号），参数也为int（圆括号）
    mypt(10);
    std::future<int> result = mypt.get_future(); //使用get_future，然后就可以使用get获得返回值了
    cout << result.get() << endl;

    system("pause");
    return 0;
}
```
- 这种写法没有创建多线程，相当于函数直接调用

## std::promise
同样也是一个类模板，我们能够在某个线程中给它复制，然后在其他线程中，把值取出来使用
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
void mythread(std::promise<int> &tmpp , int calc)
{
    //做一系列操作
    calc++;
    calc *= 10;
    std::chrono::millisecond dura(50000); //休息五秒
    std::this_thread::sleep_for(dura);
    int result = calc;
    tmpp.set_value(result); //保存结果
}
int main()
{
    std::promise<int> myprom; //对象保存的值为int类型
    std::thread t1(mythread,std::ref(myprom),180);
    t1.join();
    std::future<int> ful = myprom.get_future();
    cout << ful.get() << endl;

    system("pause");
    return 0;
}
```

使用promise进行存储值，在多线程中使用set_value进行存储，同样有get_future进行获得future成员，并且使用get进行获取值，但是get只能调用一次，不能调用多次，否则必然报错
- 只要用了thread对象，就一定要join，不然容易报错

# future其他成员函数
## std::future_status
这个函数可以设置等待时间，通过future的wait_for去判定一个线程是否符合自己所需要等待的时间，std::future_status::ready,std::future_status::timeout,std::future_status::differed，其中如若async不指定第一个参数，默认返回结果为differed.

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
int mythread() //线程入口函数
{
    cout << "mythread start " << "Thread id::" << std::this_thread::get_id() << endl;
    std::chrono::milliseconds dura(5000); //休息五秒
    std::this_thread::sleep_for(dura);
    cout << "mythread end" << endl;
    return 5;
}
int main()
{
    //cout << "main : " << std::this_thread::get_id() << endl;
    std::future<int> result = std::async(std::launch::async,mythread);
    //枚举类型
    std::future_status status = result.wait_for(std::chrono::seconds(1)); //等待线程是否超时
    if (status == std::future_status::timeout)
    {
        //超时表示线程还没结束完 
        cout << "Time limit" <<  endl;
    }
    else if (status == std::future_status::ready)
    {
        cout << "Ready" << endl;
        result.get();
    }
    else if (status == std::future_status::deferred)//延迟
    {
        cout << "Differed" << endl;
    }

    system("pause");
    return 0;
}
```

## std::shared_future
传统future只能被get一次，而使用移动语义将future过渡到std::shared_future之后，则可以被get多次
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <mutex>
using namespace std;
int mythread(int num) //线程入口函数
{
    cout << "mythread start " << "Thread id::" << std::this_thread::get_id() << endl;
    cout << "mythread num : " << num << endl;
    std::chrono::milliseconds dura(5000); //休息五秒
    std::this_thread::sleep_for(dura);
    cout << "mythread end" << endl;
    return 5;
}
int main()
{
    //cout << "main : " << std::this_thread::get_id() << endl;
    std::packaged_task<int(int)> mypt(mythread);
    std::thread t1(std::ref(mypt),1);
    t1.join();
    
    std::future<int> result = mypt.get_future();
    std:shared_future<int> result_s(std::move(result));
    //或直接使用get_future获得对象
    //std:shared_future<int> result_s(mypt.get_future());
    cout << result_s.get() << endl;
    cout << result_s.get() << endl;

    system("pause");
    return 0;
}
```

# 原子操作std::atomic
原子操作：多线程中不会被打断的程序片段，且效率比互斥量效率更进一筹。即要么完成，要么未完成，不可能出现半完成状态
- 原子操作一般针对一个变量而不是一个代码段

使用std::atomic类模板，生成对象，且可以像操作普通对象一样操作它
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>
std::atomic<int> g_count(0);
//int g_count = 0;
void mythread()
{
    for (int i = 0 ; i < 10000000 ; i++)
    {
        g_count++;
    }
}
using namespace std;
int main()
{
    std::thread t1(mythread);
    std::thread t2(mythread);

    t1.join();
    t2.join();
    cout << g_count << endl;

    system("pause");
    return 0;
}
```

## 叙谈std::atomic
并不是所有运算符都支持原子操作的，一般atomic原子操作针对++,--,+=,&=,|=,^=,其他的可能不支持。如之前的代码改为g_count = g_count + 1，返回的结果就出问题了

# windows临界区
临界区可以近似看成mutex，代码如下：
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <list>
#include <mutex>
#include <windows.h>

#define __WINDOWSJQ_

using namespace std;

class A
{
private:
    list<int> myData;
    std::mutex my_mutex;
#ifdef __WINDOWSJQ_
    CRITICAL_SECTION my_winsec; //windows临界区，近似于互斥量
#endif

public:
#ifdef __WINDOWSJQ_
    A()
    {
        InitializeCriticalSection(&my_winsec); //初始化临界区
    }
#endif

    void pushData()
    {
        cout << "Enter pushData " << endl;
        for (int i = 0 ; i < 100000 ; i++)
        {
#ifdef __WINDOWSJQ_
            EnterCriticalSection(&my_winsec);
            myData.push_back(i);
            LeaveCriticalSection(&my_winsec);
#else

            my_mutex.lock();
            myData.push_back(i);
            my_mutex.unlock();
#endif
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
#ifdef __WINDOWSJQ_
                EnterCriticalSection(&my_winsec);
                myData.pop_front();
                LeaveCriticalSection(&my_winsec);
#else
                my_mutex.lock();
                myData.pop_front();
                my_mutex.unlock();
#endif
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
在同一个线程，可以多次进入临界区，但是相应的也需要多次退出临界区。但是使用mutex不能多次加锁

# 自动析构技术
RAII类（Resource Acquisition is initialization） 资源获取即初始化
- 即构造函数进行锁操作，析构函数进行解锁，在括号体内直接声明对象即可

# recursive_mutex递归独占互斥量
- std::mutex 独占互斥量，自己lock时，别人就无法lock
- recursive_mutex 递归独占互斥量，允许多次lock，解决多次调用的问题
- timed_mutex 带超时的互斥量，如果在一定时间内拿不到锁，流程继续往下走，拥有函数try_lock_for，以及try_lock_until来尝试获取锁
- recursive_timed_mutex 递归带超时的互斥量

# 补充知识点
## 虚假唤醒
如果多次调用notify_one，则可能造成虚假唤醒，因此可以在wait中第二个参数加入lambda函数，来防止虚假唤醒
```cpp
my_cond.wait(sbguard1,[this]{
                if (!myData.empty())
                    return true;
                return false;
            });
```

## 浅谈线程池
- 如果业务有需求，线程数量尽量不要超过500个，能控制在200个以内