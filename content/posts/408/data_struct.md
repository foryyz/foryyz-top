---
title: 'DataStruct - 数据结构'
date: 2024-11-25T18:06:22+08:00
draft: false
tags: ['C','408']
---

# 数据结构

## 1 数据结构基本概念

数据：

- 数据对象 (具有**相同性质**的数据元素的集合)
  - 数据元素 
    - 数据项
- 数据结构 (相互之间存在一种或多种特定关系的**数据元素的集合**)
- 数据类型、抽象数据类型
  - 原子类型 - 其值不可再分
  - 结构类型 - 其值可以再分解成若干分量



### 1.1 数据结构三要素

1. 逻辑结构 **(*定义)**
   - 集合结构
   - 线性结构 - 一对一 。
     - 有开头，有结尾
   - 树形结构 - 一对多
   - (网)图状结构 - 多对多
2. 数据的运算
   - 增删改查
3. 物理结构(存储结构) **(*实现)**
   - **顺序存储**
   - 非顺序存储
     - **链式存储**
     - **索引存储** - 存储元素信息的同时，建立附加的索引表
     - **散列存储** - Hash存储

数据的存储结构会影响存储空间分配的方便程度，也会影响对数据运算的速度

![](../assets/data_struct/1.1_1.png)

### 1.2 算法 Algorithm

算法：对特定问题求解步骤的一种描述，它是指令的有限序列

​	程序 = 数据结构 + 算法

#### 1.2.1 算法的特性

- 有穷性 - 一个算法必须在执行有穷步后结束，且每一步都在有穷时间内完成
- 确定性 - 算法中每条指令必须有确切的含义，对于相同的输入必须得到相同的输出
- 可行性 - 算法中描述的操作都可以通过已经实现的基本运算执行有限次来实现。
- 输入 - 一个算法有**零个或多个**输入
- 输出 - 一个算法有**一个或多个**输出

#### 1.2.2 好算法的特性

- 正确性 - 算法应能够正确的解决问题
- 可读性
- 健壮性
- 高效率与低存储量需求

#### *1.2.3 算法效率的度量

##### 时间复杂度

​	事先预估算法**时间开销T(n)**与**问题规模n**的关系

大O表示法

- T1(n) = O(n)
- T2(n) = O(n^2)
- T3(n) = O(n^3)

复杂度量级排序：**常对幂指阶** 

![image-20241125184429729](../assets/data_struct/1.2_1.png)

- 顺序执行的代码只会影响常数项，可以忽略
- 只需挑**循环中的一个基本操作**分析它的执行次数与n的关系即可
- 如果有多层嵌套循环，只需关注最深层循环 循环了几次

![image-20241125185824828](/assets/data_struct/1.2_2.png)

##### 空间复杂度

![image-20241126151214824](../assets/data_struct/1.2_3_2.png)

![image-20241126151841651](../assets/data_struct/1.2_3_3.png)



## 2 线性表

逻辑结构上 a1-a2-a3-a4-a5

物理结构上分为：

- 顺序存储
- 链式存储

### 2.1 定义和基本操作

线性表是具有**相同数据类型**的**n**(n>=0)个**数据元素**的**有限序列**。
L = (a1, a2, ... , ai, a(i+1), ... , an )

注意：数据元素的**位序从1开始**

### 2.2 顺序表

顺序表的特点：

- 随机访问，可以在O(1)时间内找到第i个元素 -> data[i-1]
- 存储密度高，每个节点只存储数据元素
- 要求大片连续空间，拓展容量不方便
- 插入、删除操作不方便，需要移动大量元素

代码实现：

​	**[List.cpp](https://github.com/foryyz/Programming-Basic-Projects/blob/main/Data_Struct_408/List.cpp)**

### 2.3 单链表

单链表的特点：

- 优点：不要求大片连续空间，改变容量方便
- 缺点：不可随机存取，要耗费一定空间存放指针

考点：

- **[头插法逆置单链表](https://github.com/foryyz/Programming-Basic-Projects/blob/main/Data_Struct_408/LinkList.cpp#L155)**

代码实现：

​	**[LinkList.cpp](https://github.com/foryyz/Programming-Basic-Projects/blob/main/Data_Struct_408/LinkList.cpp)**