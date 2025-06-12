---
title: 'Flume 安装与基础文件采集'
date: 2025-05-20T22:00:20+08:00
draft: false
tags: ['BigData','Hadoop','Flume']
---

# FLUME



## 1 下载与安装 Flume

​	(一般将Flume安装在日志服务器下，该文档演示安装在namenode服务器下)

Flume下载地址 [Download — Apache Flume](https://flume.apache.org/download.html)
	下载[Apache Flume binary (tar.gz) 的 未加密版本](https://dlcdn.apache.org/flume/1.11.0/apache-flume-1.11.0-bin.tar.gz)即可

下载后上传至/opt目录

```bash
# 解压
tar -zxvf /opt/apache-flume-1.11.0-bin.tar.gz -C /usr/local/software/
# 重命名
mv /usr/local/software/apache-flume-1.11.0-bin/ /usr/local/software/flume
```

配置环境变量
将下面内容添加到/etc/profile.d/my_env.sh

```bash
vim /etc/profile.d/my_env.sh

export FLUME_HOME=/usr/local/software/flume
export PATH=$PATH:$FLUME_HOME/bin
```

使用 `source /etc/profile.d/my_env.sh` 命令激活



## 2 配置 Flume

配置文件目录 /usr/local/software/**flume/conf/**

复制模板文件 -> flume-env.sh

```bash
cd /usr/local/software/flume/conf/
cp flume-env.sh.template flume-env.sh
```

修改配置文件中的 JAVA_HOME 值

```
# 添加
export JAVA_HOME=/usr/local/software/jdk
```



## 3 使用 Flume 进行流式数据采集

Flume的使用思路：使用Flume 从 数据目录 采集 源数据，经过通道处理，再通过输出通道与HDFS连接

执行流程：
	①收集数据集放入/opt/logs目录
	②编写Flume配置文件 flume-dir-hdfs.conf (本例子存储到/usr/local/software/flumeconfig) 

**flume-dir-hdfs.conf**

```conf
# 配置agent的名字与三大组件
# 默认agent的名字是a1
a1.sources = r1
a1.sinks = k1
a1.channels = c1


# 配置source组件的内容
# 监听目录的方式: spooldir
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /opt/logs
a1.sources.r1.fileHeader = false

# 配置sink组件的内容
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoopCluster/jobdata/%Y%m%d/%H
a1.sinks.k1.hdfs.filePrefix = job
a1.sinks.k1.hdfs.useLocalTimeStamp = true
a1.sinks.k1.hdfs.fileType = DataStream

# 配置channel组件的内容
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

 # 将三个组件整合起来
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

### 执行flume文件

```bash
flume-ng agent -c conf -f /usr/local/software/flumeconfig/flume-dir-hdfs.conf -n a1
```

