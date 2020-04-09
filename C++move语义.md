---
title: c++ move语义
date: 2019-04-24 20:45:43
tags:
- 编程
- 学习
- C家族
- 多线程
categories : "C家族"
---

> - c++11新特性
> - c++11 move 移动语义
> - 左值 右值等概念

<!--more-->

---
# 左值右值
在C++11中拥有左值和右值的概念，但是可以近似理解成，可以取地址的变量为左值，反之为右值（类似于临时变量），如下：
```
int a = 10; //在其中，a为左值，10为右值
vector<int>(10); //右值
```

# 移动语义
移动语义作为新特性引入，主要有两个目的：
- 消除两个对象不必要的拷贝，提高效率
- 能够更加简洁明确的定义泛型函数

## 代码示例
首先来看以下例子：
```cpp
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;
class Test
{
private:
    int* num;
public:
    Test() : num(new int[101]) {} //构造函数
    Test(const Test& t) //拷贝构造
    {
        num = new int[101];
        memcpy(num,t.num,sizeof(int)*101); //内存拷贝
    }
    ~Test() //析构函数
    {
        delete[] num;
    }
};

Test createTest()
{
    return Test();
}

int main()
{
    Test t1;
    Test t2(t1);
    Test t3(createTest());
    return 0;
}
```
通过上面的例子，我们可以看到在main函数当中，t2调用了拷贝构造。如果t1和t2都有各自的用途，那么上面的代码没什么问题。

但是t3不一样。如果我们使用这种写法，在t3当中，我们的拷贝构造将被调用两次，即createTest函数内部返回时调用了一次，而传入t3时又调用了一次。

但是这种是十分没必要的，如若数组空间极大，那么效率极低，因此move函数应运而生。来看以下写法：
```cpp
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;
class Test
{
public:
    int* num;
public:
    Test() : num(new int[20]) 
    {
        cout << "Constructor" << endl;
        for (int i = 0 ; i < 20 ; i++)
            num[i] = i;
    } //构造函数
    Test(const Test& t) //拷贝构造
    {
        cout << "Copy Constructor" << endl;
        num = new int[20];
        memcpy(num,t.num,sizeof(int)*20); //内存拷贝
    }
    Test(Test&& t) : num(t.num) {}
    ~Test() //析构函数
    {
        delete[] num;
    }
    void show()
    {
        for (int i = 0 ; i < 20 ; i++)
            cout << num[i] << " ";
        cout << endl;
    }
};

Test createTest()
{
    return Test();
}

int main()
{
    Test t1;
    Test t2(move(t1));
    t1.num[3] = 15;
    t1.show();
    t2.show();

    return 0;
}
```

结果输出如下：
```
Constructor
0 1 2 15 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
0 1 2 15 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
```

可以观察到，如果我们修改t1中的num[3]，在t2中的结果也会跟着改变，说明t2中的num已经指向了t1中的num，但是这种写法会引发错误，即重复析构，因此我们需要修改Move构造，将t1的num赋值为空。修正后的代码如下：
```cpp
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;
class Test
{
public:
    int* num;
public:
    Test() : num(new int[20]) 
    {
        cout << "Constructor" << endl;
        for (int i = 0 ; i < 20 ; i++)
            num[i] = i;
    } //构造函数
    Test(const Test& t) //拷贝构造
    {
        cout << "Copy Constructor" << endl;
        num = new int[20];
        memcpy(num,t.num,sizeof(int)*20); //内存拷贝
    }
    Test(Test&& t) : num(t.num) 
    {
        cout << "Move Constructor" << endl;
        t.num = NULL;
    }
    ~Test() //析构函数
    {
        delete[] num;
    }
    void show()
    {
        for (int i = 0 ; i < 20 ; i++)
            cout << num[i] << " ";
        cout << endl;
    }
};

int main()
{
    Test t1;
    Test t2(move(t1));
    t2.show();
    return 0;
}
```

# 完美转发
首先查看如下代码：
```cpp
#include <iostream>
using namespace std;

void F(int a) {
	cout << a << endl;
}

template<class A>
void G(A &&a) {
	return F(a);
}

int main() {
	G(5);

    return 0;
}
```

在这段代码当中，先看G函数，我们使用数字5作为传入参数，在函数内部中，继续调用F函数，理论上来说，传入的参数依然是个右值，但是事实并非如此，编译器认为它依然是个左值，因此我们需要使用完美转发，代码如下：
```cpp
#include <iostream>
using namespace std;

void F(int a) {
	cout << a << endl;
}

template<class A>
void G(A &&a) {
	return F(std::forward<A>(a));  
}

int main() {
	G(5);

	system("pause");
    return 0;
}
```