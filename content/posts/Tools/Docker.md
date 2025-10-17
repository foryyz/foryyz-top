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

方式1.

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

方式2.

MacOS, 安装mysql8

在某位置创建 `docker-compose.yml`

```yml
# 新版本的docker-compose可以删除这一行
version: '3.9'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

| **操作**             | **命令**                                 |
| -------------------- | ---------------------------------------- |
| 启动 MySQL           | docker-compose up -d                     |
| 停止 MySQL           | docker-compose down                      |
| 查看日志             | docker logs mysql                        |
| 进入 MySQL 命令行    | docker exec -it mysql mysql -u root -p   |
| 删除容器但保留数据   | docker-compose down                      |
| 删除数据（彻底清空） | docker volume rm docker-mysql_mysql_data |
