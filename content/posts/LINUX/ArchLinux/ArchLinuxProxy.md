---
title: 'ArchLinux Proxy - Arch的代理设置方法'
date: 2024-06-06T09:37:00+08:00
draft: false
tags: ['Linux','ArchLinux','Proxy']
---
> Author - yyz
>
> Create Time - 2024/5/22
>
> **Last Update Time - 2024/5/23**

# Linux终端代理的实现方法

操作系统 - **ArchLinux**，其他linux系统的实现原理也相通。

支持终端 ↓

- bash,zsh
- fish

## 方式一、

```shell
#在对应的 ~/.bashrc 或 ~/.zshrc 或 ~/.config/fish/config.fish 中添加：
export http_proxy=http://127.0.0.1:xxxx
export https_proxy=http://127.0.0.1:xxxx
export all_proxy=socks5://127.0.0.1:xxxx
```

这种实现方式会将运行的**所有命令都走代理**,并不优雅,所以使用**alias**命令为这些命令设置别名.

## 方式二、(推荐)

对于**bash**和**zsh**.**↓**

```shell
# 定义代理地址变量
httpproxy=http://127.0.0.1:xxxx
socksproxy=socks5://127.0.0.1:xxxx

# 设置使用代理
alias setproxy="export http_proxy=$httpproxy; export https_proxy=$httpproxy; export all_proxy=$socksproxy; echo 'Set proxy successfully'"

# 设置取消使用代理
alias unsetproxy="unset http_proxy; unset https_proxy; unset all_proxy; echo 'Unset proxy successfully'"

# 查ip
alias ipcn="curl myip.ipip.net"
alias ip="curl ip.sb"
```

对于**fish**. **↓**

```shell
# 定义代理地址变量(两个端口号需要改成你自己的)
set httpproxy http://127.0.0.1:xxxx
set socksproxy socks5://127.0.0.1:xxxx

# 设置使用代理
alias setproxy="export http_proxy=$httpproxy; export https_proxy=$httpproxy; export all_proxy=$socksproxy; echo 'Set proxy successfully'"

# 设置取消使用代理
alias unsetproxy="set -e http_proxy;set -e https_proxy;set -e all_proxy;echo 'Unset proxy successfully'"

# 查ip
alias ipcn="curl myip.ipip.net"
alias ip="curl ip.sb"
```

source配置文件后，就可以使用以下命令了:

- **setproxy** - 开启代理 , **unsetproxy** - 关闭代理
- **ipcn** - 查询ip属地 , ip - 查询ip

> [!NOTE]
>
> 终端的每个标签都是独立的，**setproxy命令只会为当前终端设置代理**。新建的标签是不会走代理的。



# 透明代理

操作系统 - **ArchLinux**

- v2ray-core
- v2raya - https://v2raya.org/

```bash
sudo pacman -S v2ray v2raya
sudo systemctl enable --now v2raya #启动v2raya服务

#也可以在aur库安装
#paru -S v2raya-bin

```



[^参考文章]: https://www.xiebruce.top/1061.html#i-2
