---
title: 'Cpp相关的错误解决方法'
date: 2024-10-25T23:42:37+08:00
draft: false
tags: ['C++']
---

## 1 VS2022 中文乱码

### 方式1. 统一更改为utf-8

```c++
// 添加代码 设置控制台编码为utf-8
system("chcp 65001");
```

### 方式2.命令行设置/utf-8

vs2022菜单栏 - 项目 - 属性 - 配置属性 - C/C++ - 命令行

加上 `/utf-8`

