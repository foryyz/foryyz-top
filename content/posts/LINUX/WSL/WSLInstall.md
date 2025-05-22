---
title: 'Wsl - 2025年的WSL2安装教程'
date: 2024-11-13T18:59:19+08:00
draft: false
tags: ['Wsl','Linux']
---

# WSL2 安装教程 2025

虽然2025年还没到，但这是**2024-11-13**更新的教程，约等于2025（:

## 1 启动功能

控制面板 - 程序 - 启动或关闭Windows功能

打开：**Windows 虚拟机监控程序平台** 和 **适用于Linux的Windows子系统**

## 2 安装Linux内核更新包

下载 [适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) 安装并重启

重启后在终端执行：`wsl --set-default-version 2`

`dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`

## 3 安装Linux系统

在Microsoft Store中，安装Ubuntu，版本自己选

