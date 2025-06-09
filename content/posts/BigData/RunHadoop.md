---
title: 'Runner 启动集群'
date: 2024-06-22T22:41:20+08:00
draft: false
tags: ['BigData','Hadoop','Hive']
---

# Runner

集群昵称 - hadoopCluster

## 启动

**hadoop101、hadoop102、hadoop103**

```bash
# 开启zookeeper服务
zkServer.sh start
# 开启journalnode服务
hdfs --daemon start journalnode
```

**hadoop101**

```bash
# 启动集群
start-all.sh
# 启动历史服务器
mapred --daemon start historyserver

# 启动spark
start-spark.sh
# 启动历史服务器
start-history-server.sh
```

**Hive** -**hadoop101**

```bash
# 终端 1
hive --service metastore
# 终端 2
hive --service hiveserver2
# 远程连接
beeline -u jdbc:hive2://192.168.170.101:10000 -n root
```



## 关闭

**hadoop101**

```bash
stop-history-server.sh
stop-spark.sh

mapred --daemon stop historyserver
stop-all.sh
```

**hadoop101、hadoop102、hadoop103**

```bash
zkServer.sh stop
```

