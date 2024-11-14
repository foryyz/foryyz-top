---
title: 'AutoDeployer_Cpp - Windows环境部署工具'
date: 2024-10-21T21:37:19+08:00
draft: false
tags: ["C++","yyz"]
---

> 项目开始时间: 2024/10/08 (a)
>
> 项目成员: yyz
>
> 最新更新时间: 2024/11/14 (b)
>
> 版本号: b0.101

# AutoDeployer_Cpp

**b0.101(第二迭代版本)** 开源python版本请前往：[[GitHub - foryyz/AutoDeployer]](https://github.com/foryyz/AutoDeployer)

**项目目的：实现一键的全自动Windows环境部署**

**当前版本支持部署模式：环境变量式 - env_key**



## 项目说明

### 应用场景：

- 大规模批量的自动化环境部署
- 个人使用，规范化个人开发环境

### 开发进展：

- 详细可看[更新日志](#更新日志)

### 更新日志：

```
b0.101	2024/11/14	yyz	*Push
	- 修复 不主动创建文件夹的Bug
	- 修复 选择安装环境时的数字判断逻辑
		> 添加-退出程序选项 9;
	
b0.100	2024/11/14	yyz	*Push
	- 实现 加载配置文件，下载、解压 压缩包
```



## C++ 环境

标准: C++20

- curl
- yaml-cpp
- libarchive
