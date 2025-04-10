---
title: 'AutoDeployer_Cpp - Windows环境部署工具'
date: 2024-10-21T21:37:19+08:00
draft: false
tags: ["C++","yyz"]
---

> 项目开始时间: 2024/10/08 (a)
>
> 最新更新时间: 2024/11/16 (b)
>
> 版本号: b0.115
>
> 项目成员: yyz

# AutoDeployer_Cpp

**b0.1xx(第二迭代版本)** 开源python版本请前往：[[GitHub - foryyz/AutoDeployer]](https://github.com/foryyz/AutoDeployer)

**项目目的：实现一键的全自动Windows环境部署**

**当前版本支持部署模式：环境变量式 - env_key**



## 项目说明

### 应用场景：

- 大规模批量的自动化环境部署
- 个人使用，规范化个人开发环境

### 开发进展：

- 详细可看[更新日志](#更新日志)



## C++ 环境

标准: C++20

- libcurl
- yaml-cpp
- libarchive



## ！注意事项：

- 读取PATH时的缓存区设置为默认大小(32767字符)，如果遇到非常大的路径，可能需要处理溢出的情况
  - `wchar_t buffer[32767];`
- 下载文件时单线程最多支持int格式容纳的最大字节数据
  - 约2047MB




## 更新日志：

```
b0.115-4	2024/11/16	yyz	*Push
	- 更新 PATH路径添加时改为绝对路径(对路径中的首字符%进行识别)
	- 优化 输出格式
	= Installation Function Implementation.

b0.113	2024/11/16	yyz
	- 优化 下载算法，添加重定向并模拟浏览器头部，防止下载数据不全
	- 修复 压缩失败的return值错误

b0.112	2024/11/16	yyz	*Push
	- 修复 EnvByOneKey 安装方法HOMEPATH值错误
	
b0.111	2024/11/15	yyz
	- 修复 PATH路径添加时的字符串拼接错误
	
b0.110	2024/11/15	yyz
	- 实现 环境变量的添加实现(User级)
	- 实现 PATH环境变量的添加实现(User级)

b0.102	2024/11/14	yyz
	- 优化 EnvByKeys成员方法改为父类Env成员方法,所有子类可调用
	
b0.101	2024/11/14	yyz	*Push
	- 修复 不主动创建文件夹的Bug
	- 修复 选择安装环境时的数字判断逻辑
		> 添加-退出程序选项 9;
	
b0.100	2024/11/14	yyz	*Push
	- 实现 加载配置文件，下载、解压 压缩包
```

## IMG

![b0.115-1](../assets/AutoDeployer/b0.115-1.png)

