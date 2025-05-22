---
title: 'Compiler - 编译与链接'
date: 2024-10-25T22:09:05+08:00
draft: false
---

# Compiler - 编译与链接

## 1 编译

**含义：**将源文件 使用编译器 编译为机器语言

\*.cpp → \*.obj(Windows) / \*.o(Linux)

此时 对于**标准库**中的函数或者**模块外**的函数 只是声明了并未实际定义，需要**链接**来帮助找到这些函数

## 2 链接 (Linking)

\*.obj(Windows) / \*.o(Linux) → 可执行文件(\*.exe)

## 3 makefile

![image-20241025221857919](../asstes/Compiler/image-20241025221857919.png)

实际上是一个**依赖树**的说明文件



## **参考文献：**

[ 0 ] [隐藏的细节：编译与链接](https://www.bilibili.com/video/BV1TN4y1375q)
