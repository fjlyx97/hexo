---
title: cmake的使用
date: 2018-11-04 16:42:06
tags:
- 学习
- 编程
- C家族
- makefile 
categories : "C家族"
---


> - 使用开源框架SeetaFaceEngine需要手动改写CMakeList.txt
> - 简要入门cmake的使用

<!--more-->

# 单个源文件
- 目录树如下：
```
cpp
├── build
├── CMakeLists.txt
└── main.cpp
```
- 源文件放置于根目录，进行CMakeLists.txt的编写
```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Hello)
# 指定生成目标
add_executable(Hello main.cpp)
```
- 切换到目录build，使用cmake .. && make进行编译，即可得到可执行文件Hello

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Hello)
# 指定生成目标
# 这里指定目录，得到变量
# 格式：aux_source_directory(<dir> <variable>)
aux_source_directory(. DIR_SRCS)

add_executable(Hello ${DIR_SRCS})
```


# 多个目录多个源文件
- 现有目录如下：
```
cpp
├── CMakeLists.txt
├── include
│   └── temp.h
└── src
    ├── main.cpp
    └── temp.cpp
```
- temp中含有一个say函数，再main函数中进行调用，编写CMakeLists如下：
```cmake
cmake_minimum_required(VERSION 2.6)

project(hello)

#引入头文件
include_directories(./include)

#导入目录
aux_source_directory(. PROJECT_ROOT)
aux_source_directory(./src SRC)

add_executable(hello ${PROJECT_ROOT} ${SRC})
#生成共享库
add_library(libhello SHARED ${PROJECT_ROOT} ${SRC})
```

# 常用编译选项
```
#设置工程所生成的目标文件所需要的链接的库，在这里我们需要libhello.so这个库
target_link_libraries (${PROJECT_NAME} libhello)

#add_definitions   设置编译选项
add_definitions ("-g")   #-g编译成可调试的程序

#设置编译好的可执行文件在执行时连接库的路径
set (CMAKE_INSTALL_RPATH .;/usr/local/lib64)
set (CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

add_subdirectory  #将工程中的子目录加入编译工程中,子目录当中需要CMakeLists.txt文件
```

