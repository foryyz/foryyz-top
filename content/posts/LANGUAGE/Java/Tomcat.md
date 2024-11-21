---
title: 'Tomcat'
date: 2024-11-17T17:26:07+08:00
draft: false
tags: ['Java','Web']
---

# Tomcat

## x0 一些错误的解决

### x0.1 控制台中文乱码

​	以下方式选一种就行，优先第二种

①在tomcat目录下，找到conf文件夹里的**logging.properties**
找到并修改为`java.util.logging.ConsoleHandler.encoding = GBK`

②idea中，虚拟机选项中添加参数 `-Dfile.encoding=UTF-8`
