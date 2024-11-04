---
title: 'CMake - 运行原理和使用流程'
date: 2024-10-25T19:18:10+08:00
draft: false
tags: ['C++']
---

# CMake

**CMake是一门**为C++服务的**语言**,即 CMake Language

[CMake和Vcpkg的集成](../Vcpkg)



## 1 CMake 项目流程

[CMakeLists.txt详解](#15CMakeLists.txt详解)

### 1.1 创建Cpp源码文件

**hello.cpp**

```c++
#include <iostream>

int main(){
    std::cout << "Hello" << std::endl;
    return 0;
}
```

### 1.2 创建CMake配置文件

**CMakeLists.txt**

```cmake
#这三项包含了配置文件最基本的信息
#支持的cmake最小版本
#项目昵称
#添加可执行文件

cmake_minimum_required(VERSION 3.20)
project(Hello)
add_executable(Hello hello.cpp)
```

### 1.3 指定CMake项目生成目录

```shell
cmake -B build

#注-可以通过cmake -G 来指定C++解释器
```

### 1.4 生成二进制可执行文件

```shell
cmake --build build
```

此时，若是Windows系统，则./build/Debug文件夹下就生成了可执行文件**hello.exe**

### 1.5 CMakeLists.txt详解

```cmake
# 寻找第三方库,REQUIRED参数 表示 该库是必须的
find_package(pkg_name REQUIRED)

# 链接第三方库
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE pkg_name)

# 打开C++标准支持，这里是C++17
target_compile_features(${CMAKE_PROJECT_NAME}) PRIVATE cxx_std_17

# 将<项目根目录>/assets 拷贝到 <项目根目录>/build/Debug/assets
add_custom_commad(
	TARGET ${CMAKE_PROJECT_NAME}
	POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory
		"${PROJECT_SOURCE_DIR}/assets"
		"$<TARGET_FILE_DIR:${CMAKE_PROJECT_NAME}>/assets")

```



## 2 CMake 程序的执行

### 2.1 cmake的命令行工具

- cmake
- ctest
- cpack
- cmake-gui
- ccmake

### 2.2 如何不使用CMakeLists.txt运行CMake

创建.cmake文件

```shell
touch first.cmake
```

**first.cmake**

```cmake
message("hello")
```

执行.cmake文件语法 - `cmake -P <file_name>`

```shell
cmake -P first.cmake

>>>输出hello
```

### 2.3 打印 - message("")

```cmake
#单行打印
message("hello")
message(hello)

#多行打印
message("first line
second line")
```

### 2.4 获取信息 - ${}

```cmake
#使用${}指定变量
message(${CMAKE_VERSION})
```



## 3 CMake 语言语法

### 3.1 变量操作

#### set()

```cmake
#方式1 - 设置单个值
set(var1 hello)
message(${var1})

#方式2 - 设置多个值 列表
set(var2 a1 a2) # 等效于set(var2 a1;a2)
#cmake的列表间元素用;分割，但是打印时不显示;
message(${var2})

```

#### ${}

```cmake
#可以使用${}打印系统环境变量
message($ENV{PATH})

#作用域为该项目
set(ENV{ABC} "ENV-Test")
message($ENV{ABC})
```

#### unset()

```cmake
set(ENV{ABC} "ENV-Test")
message($ENV{ABC})
unset(ENV{ABC})

#unset后 该变量就被移除 打印时会报错
#message($ENV{ABC})
```

