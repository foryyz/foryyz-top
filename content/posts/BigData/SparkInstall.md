---
title: 'Spark&Scala 安装与配置'
date: 2024-06-22T22:41:20+08:00
draft: false
tags: ['BigData','Hadoop','Spark']
---

# SPARK



## 1 Scala 安装

官网 [The Scala Programming Language](https://www.scala-lang.org/)
下载地址 [All Available Versions | The Scala Programming Language](https://www.scala-lang.org/download/all.html)
	[2.13.16版本](https://github.com/scala/scala/releases/download/v2.13.16/scala-2.13.16.tgz)

上传到/opt目录后

```bash
# 解压
tar -zxvf /opt/scala-2.13.16.tgz -C /usr/local/software/
# 重命名
cd /usr/local/software/
mv scala-2.13.16/ scala
```

添加环境变量 `vim /etc/profile.d/my_env.sh`

```sh
# 追加以下内容
export SCALA_HOME=/usr/local/software/scala
export PATH=$PATH:$SCALA_HOME/bin
```

激活 `source /etc/profile.d/my_env.sh`

直接输入 scala 可以看出是否安装成功 (:quit 退出)



## 2 Spark 安装

**演示为 Scala 版本 Spark**

下载地址 [Downloads | Apache Spark](https://spark.apache.org/downloads.html)

### 2.1 基本安装

上传到/opt目录后

```bash
# 解压
tar -zxvf /opt/spark-3.5.5-bin-hadoop3-scala2.13.tgz -C /usr/local/software/
# 重命名
cd /usr/local/software/
mv spark-3.5.5-bin-hadoop3-scala2.13/ spark
```

配置环境变量 `vim /etc/profile.d/my_env.sh`

```sh
export SPARK_HOME=/usr/local/software/spark
export SPARKPYTHON=/usr/local/software/spark/python
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin:$SPARKPYTHON
```

激活 `source /etc/profile`

Spark 测试：

```bash
spark-shell
```

### 2.2 配置 Spark

配置文件 **spark/conf/spark-env.sh**

复制模板文件 `cp conf/spark-env.sh.template conf/spark-env.sh`

**spark-env.sh** (在文件末尾追加 ↓ 对应软件的路径和ip要自行修改)

```sh
export JAVA_HOME=/usr/local/software/jdk
export HADOOP_HOME=/usr/local/software/hadoop
export HADOOP_CONF_DIR=/usr/local/software/hadoop/etc/hadoop
export JAVA_LIBRAY_PATH=/usr/local/software/hadoop/lib/native
export SPARK_DIST_CLASSPATH=$(/usr/local/software/hadoop/bin/hadoop classpath)

export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER 
-Dspark.deploy.zookeeper.url=hadoop101:2181,hadoop102:2181,hadoop103:2181 
-Dspark.deploy.zookeeper.dir=/spark"

export SPARK_WORKER_MEMORY=8g
export SPARK_WORKER_CORE=8
export SPARK_MASTER_WEBUI_PORT=6633
```

### 2.3 集群模式安装

**-> hadoop101**

进入spark目录后

```bash
# 复制模板 workers
cp conf/workers.template conf/workers

# 复制模板 历史日志
cp conf/spark-defaults.conf.template conf/spark-defaults.conf
```

**spark/conf/workers**
**修改集群节点信息**, 清空后写入

```
hadoop101
hadoop102
hadoop103
```

**spark/conf/spark-defaults.conf**
**配置历史日志**, 追加 ↓ (hadoopCluster 应对应集群的昵称)

```
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://hadoopCluster/spark-log
```

**hadoop101**
在 hdfs中**创建spark-log目录**

```bash
hdfs dfs -mkdir /spark-log
```

**spark/conf/spark-env.sh**
**添加历史信息**, 追加 ↓ (hadoopCluster 应对应集群的昵称)

```sh
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080 
-Dspark.history.retainedApplications=30 
-Dspark.history.fs.logDirectory=hdfs://hadoopCluster/spark-log"
```

修改spark启动文件的名字 (启动文件名和hadoop的启动文件名冲突)

```bash
# 进入spark目录后
mv sbin/start-all.sh sbin/start-spark.sh
mv sbin/stop-all.sh sbin/stop-spark.sh
```

**hadoop101**
分发Spark给集群其他节点

```bash
scp -r /usr/local/software/spark/ root@hadoop102:/usr/local/software/
scp -r /usr/local/software/spark/ root@hadoop103:/usr/local/software/
```

最后需要对hadoop102和hadoop103节点的环境变量 /etc/profile 进行配置

```sh
# 在 hadoop102 和 hadoop103 的 /etc/profile.d/my_env.sh 追加下面内容
export SPARK_HOME=/usr/local/software/spark
export SPARKPYTHON=/usr/local/software/spark/python
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin:$SPARKPYTHON
```

## 3 Spark 启动和关闭

### 3.1 启动

**hadoop101**

```bash
# 启动spark
start-spark.sh

# 启动历史服务器
start-history-server.sh
```

启动后, 输入 jps 应该多出 Master、Worker、HistoryServer

启动完成后，可以通过 WebUI访问 

**Spark集群**

http://192.168.170.101:6633
http://hadoop101:6633

**Spark历史服务器**

http://192.168.170.101:18080
http://hadoop101:18080

### 3.2 关闭

**hadoop101**

```bash
stop-history-server.sh
stop-spark.sh
```

