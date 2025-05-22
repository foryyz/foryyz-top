---
title: 'Git - 基本使用方法'
date: 2024-06-06T09:39:23+08:00
draft: false
tags: ['Git']
---
> Author - yyz
>
> Create Time - 2024/06/06
>
> **Last Update Time - 2024/11/13**

# Git

## 0 基础设置

**安装**(Linux) `sudo pacman -S git`

**绑定用户名** `git config --global user.name "Username"`

**绑定邮箱** `git config --global user.email "Email"`

**生成SSH** `ssh-keygen -t rsa -C "自定义昵称"`

之后将生成的公钥文件 **id_rsa.pub** 内容复制到 **github->账户->setting->SSH and GPGkeys**

## 1 初始化仓库

```bash
#初始化仓库
git init
#添加文件
git add .
#将暂存区内容添加到仓库
git commit -m "这里是本次提交的注释"

#也可以先在github上创建仓库后直接git clone "URL" 之后再执行add和commit.这样就省去了和远程库连接的步骤
```

## 2 推送至远程仓库

```bash
#在github上创建仓库后 进行连接
git remote add origin git@github.com:foryyz/xxxx.git
#切换分支
git branch -M "分支名字"
#推送内容 -u表示关联本地和远程分支，之后的推送和拉取就可以省略
git push -u origin main
#如果地址写错了，可以解除与远程库的绑定
git remote -v
```

## 3 拉取

```bash
git clone "URL"
#或
git pull
```

## 4 修改/合并 提交

```bash
#查看git日志
git log
#查看提交文件,这里HEAD~后的数字5为要查看几个提交
git rebase -i HEAD~5
#或者直接指定从哪个id开始合并
git rebase -i "CommitID"

#将需要合并的分支前的pick改为fixup.保存
#可以再git log检查一下
git push --force
#强制提交
```

## 5 修改注释

```bash
#修改最后一次提交
git commit --amend

#修改以前提交的注释,n改为要修改的倒数第几次的记录
git rebase -i HEAD~n
#pick 改为 edit
#之后会报："非分支 正变基"，不用管
git commit --amend
#第一行修改注释
git rebase --continue
#修改完毕 可以使用git log检查
git log

#推送至仓库
git push --force origin main
```

## 6 其他基础命令

```bash
# 丢弃已修改但未暂存的所有更改
git restore .

# 清除所有未跟踪的文件（注意这会永久删除这些文件）（-f表示强制，-d表示包括目录）
git clean -fd
# 想确认一下哪些文件会被删除，可以先使用-n参数进行模拟
git clean -fdn

# 查看提交记录
git log --oneline
```

## 7 Git 代理设置

### 设置代理

```bash
# 设置http代理
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897

# 设置socks代理
git config --global http.proxy socks5://127.0.0.1:7897
git config --global https.proxy socks5://127.0.0.1:7897
# 设置一种即可
```

### 查看代理

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

### 设置传输443端口

若`git pull`时遇到 <u>ssh: connect to host github.com port 22: Connection refused</u> 的报错信息

可能是22端口被防火墙屏蔽了，可以尝试GitHub的443端口

````bash
$ vim ~/.ssh/config
```
# Add section below to it
Host github.com
  Hostname ssh.github.com
  Port 443
```
$ ssh -T git@github.com
Hi xxxxx! You've successfully authenticated, but GitHub does not
provide shell access.
````

如果`~/.ssh`目录下没有config文件，新建一个即可
