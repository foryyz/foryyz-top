---
title: 'VSCode'
date: 2024-10-25T19:18:10+08:00
draft: false
tags: ['VSCode']
---

# VSCode

## CPP

### 设置代码格式

1. 安装插件 C/C++拓展
2. 配置ClangFormat
   - 文件 > 首选项 > 设置
   - 搜索 "clang format", 找到"C_Cpp:Clang_format_style"
   - 添加 "{ BasedOnStyle: Chromium, IndentWidth: 4}"
   - 此时按 Shift+Option+F (Mac) 就可以格式化文件了
