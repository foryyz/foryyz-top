---
title: 'Brew'
date: 2025-12-09T20:05:08+08:00
draft: false
tags: ['Mac', 'Brew']
---

> Author - yyz
>
> Create Time - 2025/12/09
>
> **Last Update Time - 2025/12/09**

# Brew

## 命令

```bash
# 更新 Brew
brew update
# 显示 Brew 版本信息
brew --version

# 搜索软件包
brew search <package_name>
# 安装软件包
brew install <package_name>
# 卸载软件包
brew uninstall <package_name>
# 查看已安装的软件包
brew list

# 查看软件包信息
brew info <package_name>
# 列出过时的软件包
brew outdated
# 清理过期的软件包
brew cleanup
# 更新软件包
brew upgrade
# 查看软件包的安装路径
brew ls --full <package_name>

# 查看本地镜像源
cd "$(brew --repo)" && git remote -v
```

