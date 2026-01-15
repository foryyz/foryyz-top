---
title: 'Curl命令'
date: 2024-11-16T16:06:09+08:00
draft: false
---

> Author - yyz
>
> Create Time - 2024/11/16
>
> **Last Update Time - 2024/11/16**

# Curl

## 1 常用命令

### 1.1 HTTP常见请求

- `curl url`
  - 对url发送GET请求
- `curl -X -POST url`
  - 对url发送POST请求
    - 也可以写成`curl -XPOST url`
- `curl -XPOST url -d data`
  - 携带data数据 对url 发送 POST请求
- `curl -XPUT url -d data`
  - 放置data数据 对url 发送PUT请求

### 1.2 文件下载

- `curl -O url`
  - 在当前目录下载url内容
- `curl -o file_name url`
  - 在当前目录下载url内容并命名为file_name
- `curl --limit-rate 100k -o file_name url`
  - 限速100k 下载url内容并命名为file_name
- `curl -C - -o file_name url`
  - 恢复下载

## 

