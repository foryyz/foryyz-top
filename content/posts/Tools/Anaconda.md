---
title: 'Anaconda'
date: 2024-06-22T14:34:00+08:00
draft: false
tags: ['Python','Anaconda']
---

> Author - yyz
>
> Create Time - 2024/06/22
>
> **Last Update Time - 2024/10/30**

# Anaconda



## 0 安装Anaconda

**下载地址** - [Download Now | Anaconda](https://www.anaconda.com/download/success)

**Linux安装 ↓ (下文以2024.02-1版本为例)**

```bash
#使用wget下载安装文件
wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh

#执行安装文件
sh Anaconda3-2024.02-1-Linux-x86_64.sh
```



## 1 管理conda

```bash
#验证conda已经安装
conda -V

#更新conda至最新版本
conda update conda
conda update anaconda

#查看帮助信息
conda -h

#卸载conda
rm -rf ~/anaconda3
```



## 2 使用conda

### 环境 - 基本管理

```bash
#创建新环境 例:conda create –n env_name python=3.11 numpy pandas
conda create -n <env_name> <package_names>
conda create --prefix=/home/path python=3.11#安装虚拟环境到指定路径

#激活环境
conda activate <env_name>
#退出环境
conda deactivate

#显示已创建环境
conda env list

#删除环境
conda remove –-name <env_name> -–all
```

### 包 - 管理

```bash
#安装包
conda install <package_name>#在当前环境中安装包
conda install –name <env_name> <package_name>#在指定环境中安装包

#卸载包
conda remove <package_name>#卸载当前环境中的包
conda remove –name <env_name> <package_name>#卸载指定环境中的包

#更新包
conda update –all
conda update <package_name>#更新指定包

#将Python环境里的包导出成txt文件
pip freeze > requirements.txt
#根据requirements.txt里面的包和版本下载到本地保存
pip download -r requirements.txt -d <pack_path>

#获取当前环境中已安装的包信息
conda list
conda list -n <env_name>#精确查找

#查找可供安装的包版本
conda search <text>#模糊查找
conda search –full-name <package_full_name>#精确查找
```

### 环境 - 高级管理

```bash
#复制环境 Conda没有重命名环境功能的, 要实现这个需求可以通过克隆-删除
conda create –n <new_env_name> –clone <copied_env_name>

#分享环境 - 将当前环境下的 package 信息存入名为 environment 的 YAML 文件中
conda env export > environment.yaml

#导入环境
conda env create -f environment.yaml

#自动开启/关闭环境
conda activate   #默认激活base环境
conda activate xxx  #激活xxx环境
conda deactivate #关闭当前环境
conda config --set auto_activate_base false  #关闭自动激活状态
conda config --set auto_activate_base true  #关闭自动激活状态

```

## 3 配置Conda

```bash
#查询配置信息
conda config --show
#查看已使用哪些镜像源
conda config --get channels

#conda瘦身
#清除Conda索引缓存 - 清理没有使用过的包
conda clean -p
#删除conda保存下来的tar包。
conda clean -t
#删除所有的安装包及cache
conda clean -y --all
```

### 配置conda国内源

`conda config --show channels`  - 显示当前的镜像路径

`conda config --remove-key channels` - 删除之前添加的所有镜像源（如清华源等），恢复为anaconda默认的镜像源

#### Mac

```shell
# 配置conda-forge源
conda config --add channels conda-forge
```

#### Linux

```bash
#配置国内源 - 方法一
#添加镜像源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
#终端显示包从哪个channel下载，以及下载地址是什么
conda config --set show_channel_urls yes

#方法二 - 把下面文字拷贝到 ~/.condarc中
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud//pytorch/
show_channel_urls: true
auto_activate_base: false  #关闭自动激活状态
#其他相关
conda config --show-sources   找到.condac文件并查看里面的镜像源
```

#### windows

```powershell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/

conda config --set show_channel_urls yes #设置搜索时显示通道地址
```

### 配置conda环境和包路径(Windows)

**$PATH:**

```
C:\ENVIRONMENT\ANACONDA\anaconda3
C:\ENVIRONMENT\ANACONDA\anaconda3\Scripts
C:\ENVIRONMENT\ANACONDA\anaconda3\Library\bin
C:\ENVIRONMENT\ANACONDA\anaconda3\Library\mingw-w64\bin
```

**显示.condarc文件（如果不显示执行）**

```powershell
conda config --set show_channel_urls yes
```

**.condarc** >>>

```
envs_dirs:
  - C:\ENVIRONMENT\ANACONDA\envs
pkgs_dirs:
  - C:\ENVIRONMENT\ANACONDA\pkgs
```



## x 其他说明

## x1 如何配置pip全局镜像源

```bash
#清华源
python -m pip install --upgrade pip#更新pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

```

