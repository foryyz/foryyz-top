---
title: 'wsl - 基础使用教程'
date: 2024-06-21T00:59:24+08:00
draft: false
tags: ['Linux','Wsl']
---

> Author - yyz
>
> Create Time - 2024/06/21
>
> **Last Update Time - 2024/06/21**

# WSL2

## 1 基础语法

- 查看 `wsl --list`
- 卸载 `wsl --unregister SystemName`
- 关机 `wsl --shutdown`

## 2 进阶语法

- 导出虚拟机 `wsl --export Ubuntu-20.04 A:\your_path\Ubuntu-20.04_export.tar`
- 导入虚拟机 `wsl --import 分发版本 安装的路径 之前导出的文件`



# Ubuntu On WSL 机器学习配置

## 0 基本设置

安装后在Linux系统中使用 `nvidia-smi` 查看是否连接到物理机Nvidia驱动
更新软件列表 `sudo apt-get update`
**安装必备的工具包** `sudo apt-get install build-essential`

## 1 安装CUDA Toolkit

下载地址：[CUDA Toolkit 12.5 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-downloads)

选项如下：Linux → x86_64 → WSL_Ubuntu → 2.0 → runfile(local)

按操作执行命令：

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.5.0/local_installers/cuda_12.5.0_555.42.02_linux.run
sudo sh cuda_12.5.0_555.42.02_linux.run
```

1. 选择 accept
2. 选择 install
3. 添加环境变量

```bash
export PATH="/usr/local/cuda-12.5/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-12.5/lib64:$LD_LIBRARY_PATH"
```

检验是否安装成功：`nvcc -V`



> [【2024】Windows 11 安装WSL2并搭建深度学习环境（Docker，Pytorch）_win11 安装wsl2-CSDN博客](https://blog.csdn.net/m0_63070489/article/details/135602337)



# Spark On Ubuntu On WSL 配置

操作系统 - Ubuntu24.04 LTS

## 1 更新软件源

[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
>>>
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

之后更新软件源缓存和软件

```bash
sudo apt update
sudo apt upgrade -y
```

## 2 安装openssh-server和jdk

```bash
sudo apt-get install openssh-server openjdk-8-jdk

sudo service ssh restart
ssh localhost
# 成功进入 ssh 后，输入 exit 回车 退出。

cd ~/.ssh/
ssh-keygen -t rsa
cat ./id_rsa.pub >> ./authorized_keys
```

## 3 安装Python (可选 Anaconda)

```bash
#安装python
sudo apt install python3 python3-pip
```

### 更改pip源



## 4 安装java, hadoop和spark

```bash
# 在你下载 hadoop...tar.gz 和 spark....tgz 的目录打开终端，并进入WSL
sudo tar -xzf hadoop*.tar.gz -C /usr/local
sudo tar -xzf spark*.tgz -C /usr/local

cd /usr/local
sudo mv hadoop-* hadoop
sudo mv spark-* spark
sudo chown -R 你的用户名 hadoop
sudo chown -R 你的用户名 spark

#添加环境变量 >>>后为添加的内容
sudo nano ~/.bashrc
>>>
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export JAVA_LIBRAY_PATH=/usr/local/hadoop/lib/native
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:${SPARK_HOME}/bin
```

## 5 配置hadoop伪分布式

### HDFS 配置

**进入配置文件目录** `cd /usr/local/hadoop/etc/hadoop`

#### 修改 hadoop-env.sh

```bash
sudo nano hadoop-env.sh
#将以下内容粘贴到文件内并保存 >>>
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export HADOOP_OPTS="-Djava.library.path=${HADOOP_HOME}/lib/native"
```

#### 修改 core-site.xml

```bash
sudo nano core-site.xml
# 将以下内容粘贴到<configuration> </configuration>的中间并保存。
<property>
<name>hadoop.tmp.dir</name>
<value>file:/usr/local/hadoop/tmp</value>
<description>Abase for other temporary directories.</description>
</property>
<property>
<name>fs.defaultFS</name>
<value>hdfs://localhost:9000</value>
</property>
<property>
<name>hadoop.http.staticuser.user</name>
<value>yyz</value>
</property>
```

#### 修改 hdfs-site.xml

```bash
sudo nano hdfs-site.xml
# 将以下内容粘贴到<configuration> </configuration>的中间并保存。
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>file:/usr/local/hadoop/tmp/dfs/name</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>file:/usr/local/hadoop/tmp/dfs/data</value>
</property>
```

### HDFS 启动

```bash
#启动ssh
sudo service ssh start
#格式化Namenode
hdfs namenode -format
#启动dfs
/usr/local/hadoop/sbin/start-dfs.sh

#此时输入 jps 命令，正常会显示4个内容： Jps,NameNode,SecondaryNameNode,DataNode
#浏览器访问 http://localhost:9870 可以显示HDFS Web UI
```

### Yarn 配置

**进入配置文件目录** `cd /usr/local/hadoop/etc/hadoop`

#### 修改 mapred-site.xml

```bash
sudo nano mapred-site.xml
# 将以下内容粘贴到<configuration> </configuration>的中间并保存。
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
<property>
<name>yarn.app.mapreduce.am.env</name>
<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
<name>mapreduce.map.env</name>
<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
<name>mapreduce.reduce.env</name>
<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
```

#### 修改 yarn-site.xml

```bash
sudo nano yarn-site.xml
# 将以下内容粘贴到<configuration> </configuration>的中间并保存。
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
```

### Yarn 启动

```bash
/usr/local/hadoop/sbin/start-all.sh

#浏览器访问 http://localhost:8088/ 可以显示Yarn Web UI
```

## 6 Spark配置 启动

```bash
# 进入spark配置文件夹
cd /usr/local/spark/conf
# 编辑 spark-env.sh
sudo cp spark-env.sh.template spark-env.sh
sudo nano spark-env.sh
# 将以下内容粘贴到文件头并保存。
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)
# 启动Spark
/usr/local/spark/sbin/start-all.sh

#浏览器访问 http://localhost:8080 可以打开Spark的WebUI
```

### spark命令行

```bash
#输入spark-shell或者pyspark，可以进入Spark命令行
pyspark
```



# Ubuntu中文配置
