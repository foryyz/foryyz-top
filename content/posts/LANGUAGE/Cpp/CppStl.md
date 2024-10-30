---
title: 'C++ - STL容器'
date: 2024-10-22T22:03:40+08:00
draft: false
---

# Cpp - STL

**STL** - Strandard Template Library --标准模板库 --为了提高代码复用性

- **容器(Container)** - 数据结构
  - 序列式容器 - 强调值的排序
  - 关联式容器 - 二叉树结构
- **算法(Algorithm)** - 常用算法
  - 质变算法
  - 非质变算法
- **迭代器(Iterator)** - 容器与算法之间的胶合剂
- 仿函数 - 行为类似函数，可作为算法的某种策略
- 适配器（配接器）
- 空间配置器

## 1 String 字符串 容器

### 1.1 构造函数

**函数原型：**

- `string();`
- `string(const char* s);` // 使用字符串s初始化
- `string(const string& str);` // 拷贝构造
- `string(int n, char c);` // 使用n个c初始化

### 1.2 查找和替换

#### 查找

**find()**

函数原型：

- `int find(const string& str, int pos=0)` - 从左往右查

```c++
// find
std::string str = "abcde";
int pos = str.find("cd"); // pos = 2
int pos2 = str.find("不存在的字符"); // pos = -1
```

- `int rfind(const string& str, int pos=npos)` - 从右往左查

```c++
// rfind
std::string str = "abcdabcd";
int pos = str.rfind("d"); // pos = 7 - 即从右往左第一次出现的查找字符
int pos2 = str.rfind("不存在的字符"); // pos = -1
```

#### 替换

**replace()**

函数原型：

- `string& replace(int pos, int n, const string& str)` // **pos** 开始查找的位置，**n** 选中多少个字符，**str**替换成什么

```c++
std::string str = "abcde";
str.replace(1,2,"XXX"); // 结果为aXXXde
```

### 1.3 比较

**compare()**

- `int compare(const string &s) const;` 
  - 主要是用来比较 字符串是否相等

```c++
string str1 = "hello";
string str2 = "hello";
str1.compare(str2); // 相同为0，否则为1

str1 = "Zello";
str2 = "Aello";
str1.compare(str2); // 输出为1，即str1 > str2

str1 = "Aello";
str2 = "Zello";
str1.compare(str2); // 输出-1，即str1<str2

```

###  1.4 插入和删除

#### 插入

**insert()**

函数原型

- `string& insert(int pos ,const string& str);` // **pos**-从哪里开始插; **str**-插什么

```c++
string str = "hello";
str.insert(1,"XXX"); // 结果为 hXXXello
```

#### 删除

**erase()**

- `string& erase(int pos, int n);` // **pos**-从哪里开始删(包含pos); **n**-删多少

```c++
string str = "hello";
str.erase(1,3); // 结果为ho
```

### 1.5 子串 (切片)

**函数原型：**

- `string substr(int pos=0, int n=npos)` // **pos**-从哪里开始(包含pos); **n**-切多少

```c++
string str = "hello";
string str1 = str.substr(); // 结果为"hello" 相等于复制
string str2 = str.substr(1); // 结果为"ello" 等于从索引1开始复制
string str3 = str.substr(1,3); // 结果为"ell" 等于从1开始复制，复制3个元素
```

**实用案例：**

```c++
std::string str = "hello@qq.com";
int index = str.find('@');
std::cout << "username = " << str.substr(0,index) << std::endl;
std::cout << str.substr(index) << std::endl;

>>>
username = hello
@qq.com
```

