---
title: Effective c++ 记录
date: 2018-04-06 09:15:41
tags:
- 编程
- 学习
- C家族
categories : "C家族"
---

> - 改善程序与设计55个具体做法
> - 防止阅后遗忘
> - 再次记录易于遗忘的条款

<!--more-->

# 条款2：尽量使用const，enum,inline替换\#define

- 取一个const地址是合法的，但取一个enum，和\#define的地址就不合法。
```cpp
enum {NumTurn = 5};
//近似看成#define NumTurn 5
```

- 单纯常量：最好以const对象或enum替换\#define，对于宏，使用inline函数替换\#define。

- const出现在类型前面和类型和后面表达的意思相同，如下：
```cpp
const int a;
int const a;
``` 

- const出现在指针\*前面代表物是常量，在\*后面代表指针自身是常量。

- 如果一个变量申明有mutable，即便在const的成员函数当中，也可以修改。

- const_cast的使用
```cpp
#include <iostream>
using namespace std;
int main()
{
    /*似乎发生了常量折叠，在编译阶段，对该变量进行值替换，同时，该常量拥有自己的内存空间，并非像宏定义一样不分配空间。*/


    const int a = 20;
    int& b = const_cast<int&>(a);
    cout << b << endl;  //20
    b = 30;
    cout << b << endl;  //30
    cout << a << endl;  //20
    return 0;
}
```

# 条款4：确定对象被使用前已被初始化
- 使用local static对象替换non-local static对象，如下使用函数来唯一初始化一个对象：
```cpp
class FileSystem{...};
FileSystem& tfs()
{
    //使用static进行初始化，并返回该对象的引用
    static FileSystem fs;
    return fs;
}
//使用tfs()来获得该对象
```

# 条款6：若不想使用编译器自动生成的函数，就该明确拒绝
- 所有编译器默认生成的函数都属于public，如果不想使用他们，则应明确将其申明为private
- 使用专门的基类来防止被拷贝，如下：
```cpp
class Uncopyable
{
protected:
    Uncopyable(){}
    ~Uncopyable(){}
private:
    Uncopyable(const Uncopyable&); //参数名称可写可不写，反正不予实现。
    Uncopyable& operator=(const Uncopyable&);
};
```

# 条款7：为多态基类声明virtual析构函数
- 对于大多数人的心得而言：只有当class内含至少一个virtual函数，才为它声明virtual析构函数。
- c++并没有禁止派生的机制，所以尽量不要继承STL容器，STL容器当中析构函数并非虚析构

# 条款8：别让异常逃离析构函数
- 析构函数绝对不要吐出异常，如果产生异常将导致异常之后的代码无法执行，会导致资源泄露
- 如果需要对异常做出反应，那么class应该提供一个普通函数（非析构函数）来处理异常
- try中出现异常，try中的后继代码不执行，但是try之外的代码需要执行
- 使用如下代码将释放资源代码返回客户手上，并拥有双保险
```cpp
class DBConn
{
public:
    ...
    void close()
    {
        db.close();
    }
    ~DBConn()
    {
        if (!closed)
        {
            try
            {
                db.close();
            }
            catch (...)
            {
                //制作运转记录，记下调用close失败的原因
                ...
            }
        }
    }
private:
    DBConnection db;
    bool closed;
};
```

# 条款09：绝不在构造和析构函数中调用virtual函数
- 由于基类的构造函数会先被调用，即如果在基类构造函数中有virtual函数，调用的是基类的函数，而非派生类的函数
- 对象在基类构造函数执行之前不会成为一个真正的对象（成品）
- 析构函数一旦开始执行，所有成员变量便成为未确定的值，所以virtual也被认为是基类的函数。
- 所以确保构造函数和析构函数当中没有虚函数，哪怕是他们调用的函数也要服从该约定

# 条款10：令operator=返回一个reference to *this
- 赋值操作符尽量返回的是reference to \*this（*this的引用），如 **return \*this**
- 尽可能返回const的类型，可阻止用户因自定义类型而犯错

# 条款11：在operator=中处理“自我赋值“
- 先查看一份不安全代码
```cpp
class Bitmap{...};
class Widget
{
...
private:
    Bitmap* pb;
};
Widget& Widget::operator=(const Widget& rhs)
{
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
// 一旦使用如上代码进行自我赋值，pb将会被销毁
```
- 传统的改进方法是增加”证同测试“，加入如下语句：
```cpp
if (this == &rhs) return *this;
```

- 我们只需在复制pb时无需删除pb即可改进代码，如下
```cpp
Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap* pOrig = pb;    
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
}
```

- 所以确定任何函数操作**一个以上的对象**时候，并且多个对象是同一个对象的时候，其行为依然正确。

# 条款12：复制对象时勿忘其每一个部分
- 在任何时候只要你承担起“为派生类编写构造函数的”责任，必须保证基类所有的成分都被复制。
- 由于大多数时候基类成员为private，所以应让派生类的构造函数调用相应的基类函数。
- 在派生类中重载=，需要对基类的成分同样将进行赋值，使用如下代码：
```cpp
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
    //假设基类为Customer
    Customer::operator=(rhs);
}
```
- 在派生类构造函数当中调用copy assignment是不合法的，以至于根本没有这样的语法。

# 条款13：以对象管理资源
- 可以使用auto_ptr智能指针进行释放内存
```cpp
Investment* createInvestment();
void f()
{
    std::auto_ptr<Investment> pInv(createInvestment());   
}
// 多个auto_ptr不允许指向相同的对象
// 若是通过拷贝构造函数或使用赋值运算符复制auto_ptr，它们会变成NULL
```

- auto_ptr的替代方案为shared_ptr，如下代码：
```cpp
void f()
{
    std::tr1::shared_ptr<Investment> pInv(createInvestment());
}
//shared_ptr允许复制
```
- 但是两个指针销毁时候都是使用delete，而不是delete[]，所以不允许指向动态分配而得的数组

# 条款15：在资源管理类中提供对原始资源的访问
- shared_ptr和auto_ptr都提供一个get成员函数，可以用来显示转换，也就是可以返回智能指针内部的原始指针，假设如下代码：
```cpp
int daysHeld(const Investment* pi);
std::tr1::shared_ptr<Investment> pInv(createInvestment());
int days = daysHeld(pInv); //错误的用法，需要Investment类型的指针，不是传递shared_ptr的对象
int days = daysHeld(pInv.get()); //正确的用法
```
- shared_ptr和auto_ptr也重载了运算符->和*，可以像平时指针一样使用。

# 条款16：成对使用new和delete时要采取相应的形式
- 为了避免一些潜在错误，尽量不要对数组使用typedef，如下代码：
```cpp
typedef std::string AddressLines[4];
std::string* pal = new AddressLines;
delete pal; //错误，是一个数组
delete[] pal; //正确
```

# 条款17：以独立语句将newed对象置入智能指针
- 如果不这么做的话，一旦抛出异常，将造成指针指向未知位置，造成难以察觉的内存泄漏

# 条款18：让接口容易被正确使用，不易被误用
- 假设有如下不安全代码：
```cpp
class Date
{
public:
    Date(int month , int day , int year);
    ...
};
//如果使用如下代码调用，将会导致使用方式出错
Date d(30,3,1995);
```
- 改进了之后，代码如下：
```cpp
struct Day
{
    explicit Day(int d) : val(d) {}
    int val;
};
struct Month
{
    explicit Month(int d) : val(d) {}
    int val;
};
struct Year
{
    explicit Year(int d) : val(d) {}
    int val;
};
class Date
{
public:
    Date(const Month& m , const Day& d, const Year& y);
};
//如果使用这种方法创建就可以预防出错。
```

# 条款20：宁以pass-by-reference-to-const替换pass-by-value
- 使用引用(by reference)的方式可以避免对象切割(slicing)的问题 。

# 条款21：必须返回对象的时候，别妄想返回其reference
- 假设有如下代码：
```cpp
const Rational& operator*(const Rational& lhs,const Rational& rhs)
{
    Rational result(lhs.n * rhs.n , lhs.d * rhs.d);
    return result;
}
//一旦函数结束了之后，result被释放，返回引用将被定义为不明确的行为
```
- 再看如下代码，以下代码试图完善上文代码:
```cpp
const Rational& operator*(const Rational& lhs,const Rational& rhs)
{
    Rational* result = new Rational(lhs.n * rhs.n , lhs.d * rhs.d);
    return *result;
}
//代码看似可以完成任务，确无法决定谁对代码进行delete
```
# 条款22：将成员变量声明为private
- 如果不出意外尽量都将成员变量声明为private。
- protected并不比public更具封装性

# 条款23：宁以non-member,non-friend替换member函数
- 面对对象守则要求，数据以及操作数据的那些函数应该被捆绑在一块，这句话其实并不正确。
- 如果使用成员函数，则对类有较大的操纵权限，所以可以考虑友元函数或者普通函数进行操纵成员。
- 