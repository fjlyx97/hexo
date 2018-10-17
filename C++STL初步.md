---
title: C++STL初步
date: 2017-12-11 12:17:03
tags:
- 编程
- 学习
- C家族
categories : "C家族"
---

>- 竞赛中常用到STL模板库来提升速度
>- 在此记录平日里做题用到的常用的函数及其模板
>- 大坑待填

<!--more-->

# sort函数
- 函数包含在algorithm头文件中
- sort可以对任意对象进行排序，默认以元素从小到大进行排序
```cpp
vector <int> vect;
sort(vect.begin(),vect.end());
//或者
int num[10];
sort(num,num+10);
/* 运行自定义函数进行排序 */
bool cmp(int a, int b)
{
    return a > b;
    // a>b为true,不交换
    // a<b为false,交换
}
sort(num,num+10,cmp);

/* 允许使用其他模板进行排序 */
equal_to	相等
not_equal_to	不相等
less	        小于  
greater	        大于  
less_equal	小于等于
greater_equal	大于等于
sort(num,num+1,greater<int>());
```

# lower\_bound 以及 upper\_bound函数
- 函数包含在algorithm头文件中
- 查找第一个大于或者等于x的第一个位置
- 查找最后一个大于或者等于x的第一个位置
```cpp
int num[10];
int tmp = lower_bound(num,num+10);
```

# 不定长数组vector
- 函数包含在vector头文件中
```cpp
vector<int> a;          //申明一个向量
vector<int> a(10);      //申明一个向量包含十个元素
vector<vector<int>> a;  //申明一个二维向量
a.size();               //读取大小
a.resize();             //改变大小
a.push_back();          //向尾部添加元素
a.pop_back();           //删除尾部元素
a.clear();              //清空向量，只清空内容，不包括大小
//允许使用下标进行赋值运算等
```

# set集合
- 函数包含于set头文件中
```cpp
set <string> s;
begin()     　　   //返回set容器的第一个元素
end() 　　         //返回set容器的最后一个元素
clear()   　　     //删除set容器中的所有的元素
empty() 　　　     //判断set容器是否为空
max_size()        // 返回set容器可能包含的元素最大个数
size() 　　　     //返回当前set容器中的元素个数
rbegin()　　　　  // 返回的值和end()相同
rend()　　　　    // 返回的值和rbegin()相同
count()          // 判断是否存在，0，1
erase()          //删除元素
```

# map映射
- 函数包含于set头文件中
```cpp
map <string,int> mouth_name //表示月份名字到月份编号
mouth_name["july"] = 7;
map<string,int>::iterator it;
for ( it = mouth_name.begin (); it != mouth_name.end();++it)
{
    //允许通过it->first或者it->second来访问数据
}
```

# 栈
- 包含在stack头文件中
```cpp
push()        //将元素压入栈顶
pop()         //将元素弹出
top()         //访问栈顶元素
empty()       //判断是否栈空
size()        //返回元素个数
stack<int> s; //声明一个栈
```

# 队列
- 包含在queue头文件中
```cpp
push()        //将元素入队
pop()         //将元素出队
front()       //取队首元素但不删除
back()       //取队尾元素但不删除
empty()       //判断是否栈空
size()        //返回元素个数
queue<int> s; //声明一个队列
```
## 优先队列
```cpp
priority_queue<int> pq;     //数字越大优先级越大
pq.top()                    //访问优先级最高的元素
pq.push()                   //入队
pq.pop()                    //出队
/* 定义优先级 */
prioiry_queue <int,vector<int>,greater<int>> pq;    //数字越小优先级越大，最小堆
/* greater 大的在底下 less小的在底下
```
