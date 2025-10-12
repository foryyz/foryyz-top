---
title: 'Docker'
date: 2025-04-20T16:15:51+08:00
draft: false
tags: ['Docker']
---

# Docker

## 基础命令

### Info

```bash
# 查看version
docker --version

# 显示正在运行的容器, 参数 -a 查看所有容器
docker ps
# 查看本地所有镜像
docker images
# 查看本地所有卷
docker volume ls查看本地所有卷
# 查看所有网络
docker network ls
```

### Run

```bash
# 拉取
docker pull [name]

# 运行
docker run [name]

```



## MySQL in Docker

```bash
docker run -d \
  --name mysql57 \
  -p 3307:3306 \
  -e MYSQL_ROOT_PASSWORD=your_password \
  -v mysql57_data:/var/lib/mysql \
  mysql:5.7
  
# -d：后台运行
# --name mysql57：容器名字叫 mysql57
# -p 3307:3306：把容器的3306端口映射到你Mac本地的3307端口（因为你本地3306已经被 MySQL 8 占用了）
# -e MYSQL_ROOT_PASSWORD=your_password：设置 root 用户的密码（你可以换成你想要的密码）
# -v mysql57_data:/var/lib/mysql：使用一个叫 mysql57_data 的 volume 保存数据，容器重启数据不会丢
# mysql:5.7：指定镜像
```

