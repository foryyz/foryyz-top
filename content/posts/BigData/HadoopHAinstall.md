---
title: 'Hadoop集群 完全分布式安装 (HA)'
date: 2025-05-20T22:00:20+08:00
draft: false
tags: ['BigData','Hadoop']
---

# Hadoop集群 (HA高可用) 完全分布式安装教程

操作系统 - CentOS 7
集群昵称 - hadoopCluster

## 0 环境准备

**下载虚拟机软件和操作系统镜像**

CentOS 7 下载地址：[Index of /7.9.2009/isos/x86_64](https://vault.centos.org/7.9.2009/isos/x86_64/)
	选择[CentOS-7-x86_64-DVD-2207-02.iso](https://vault.centos.org/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2207-02.iso)下载

VMware 下载地址：[Fusion and Workstation | VMware](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)

**为VMware设置虚拟网络**
	-> 编辑 -> 虚拟网络编辑器 -> 选中VMnet8 -> 更改设置 -> NAT模式; 子网ip 改为192.168.170.0



## 1 安装第一个CentOS系统

创建新的虚拟机 -> 自定义 -> 下一步 -> 稍后安装操作系统 -> Linux, 版本选择 CentOS 7 64位
-> 下一步 -> 虚拟机昵称hadoop101 -> 下一步 -> 内存设置为1024 (或2048) -> 使用网络地址转接(NAT) -> 下一步 -> 下一步 -> 创建新虚拟硬盘 -> 20-40GB, 拆分成多个文件 -> 下一步 ->完成

点击新建的虚拟机hadoop101 -> 编辑虚拟机设置 -> 选择 CD/DVD (IDE) -> 使用ISO映像文件, 选择新下载的CentOS镜像

启动虚拟机 -> 选择Install CentOS 7 -> 选择语言(English - US) Next -> 设置时间(ShangHai)
SYSTEM区会有感叹号提示, 点击"INSTALLATION DESTINATION"后点击左上角"Done"即可
-> (可以跳过)在"NETWORK & HOST NAME"设置主机昵称Host name
-> Begin Installation 开始安装
-> 点击 "ROOT PASSWORD" 设置root用户密码
-> Reboot



## 2 配置Hostname、IP、防火墙

登录root用户

修改主机名

```bash
hostnamectl set-hostname hadoop101
```

配置网络

```bash
# ifcfg-ens33 中的 33 需要替换成实际的文件名, 大多数为33 或 32
vi /etc/sysconfig/network-scripts/ifcfg-ens33

# 修改成下面的值(没提到的不用动)
# 注意 IPADDR和GATEWAY的值需要和VMware虚拟网络配置匹配
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.170.101
NETMASK=255.255.255.0
GATEWAY=192.168.170.2
DNS1=114.114.114.114
DNS2=8.8.8.8

# 修改完成后重启网络
systemctl restart network
```

此时就可以正常连接网络了，可以 `ping baidu.com` 进行测试

更新和下载软件包

```bash
# 下载阿里云源配置文件
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 更新缓存
yum clean all
yum makecache
# 更新软件
yum update -y
# 下载必须软件
yum -y install openssh-server vim net-tools
# (可选)下载其他常用软件
yum -y install gcc gcc-c++ autoconf automake cmake make \
 zlib zlib-devel openssl openssl-devel pcre-devel \
 rsync man zip unzip tcpdump lrzsz tar wget nano
```

配置hosts

```bash
vi /etc/hosts

# 添加以下内容
192.168.170.101	hadoop101
192.168.170.102	hadoop102
192.168.170.103	hadoop103
```

关闭防火墙

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

systemctl stop firewalld
systemctl disable firewalld
```

同步时间

```bash
yum install chrony -y
# 启动服务
systemctl start chronyd
systemctl enable chronyd
# 同步时间
chronyc makestep
```



## (可选)3 创建hadoop用户

对于生产环境, 通常推荐单独创建一个hadoop用户进行权限限制, 如果只是用于学习开发可以跳过此环节
如果未跳过此环节，后面的步骤需全部使用新创建的hadoop用户执行，并将配置文件中的用户设置为hadoop

```bash
# 创建用户
sudo useradd -m hadoop
# 设置密码
sudo passwd hadoop

# 将 Hadoop 用户添加到 wheel 组
sudo usermod -aG wheel hadoop

# 为 Hadoop 用户设置权限
sudo chown -R hadoop:hadoop /opt
sudo chmod -R 755 /opt

```



## 4 安装JDK

openjdk下载地址 [Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/#java8-linux)
选择 x64 Compressed Archive下载
下载完成后 使用shell工具 将文件传输到系统的/opt中；或者直接进入/opt目录执行 `wget \<下载链接\>`

```bash
mkdir /usr/local/software

# 解压 (此处需要替换为上传的jdk压缩包)
tar -zxvf /opt/jdk-8u451-linux-x64.tar.gz -C /usr/local/software

# 更改文件夹昵称
mv /usr/local/software/jdk1.8.0_451/ /usr/local/software/jdk
```

配置环境变量

```bash
# 此处可以修改/etc/profile，下面方法可以起到隔离的作用
vim /etc/profile.d/my_env.sh

# 添加以下内容
export JAVA_HOME=/usr/local/software/jdk
export PATH=$PATH:$JAVA_HOME/bin

# 激活
source /etc/profile.d/my_env.sh

# 检查是否安装成功
java -version
```

## 5 配置Zookeeper

zookeeper下载地址 [Apache ZooKeeper](https://zookeeper.apache.org/releases.html#download)
推荐选择下载latest stable release版本 (书写该文档时为[3.8.4](https://dlcdn.apache.org/zookeeper/zookeeper-3.8.4/apache-zookeeper-3.8.4-bin.tar.gz))

上传至/opt文件夹后 ->

```bash
# 解压
tar -zxvf /opt/apache-zookeeper-3.8.4-bin.tar.gz -C /usr/local/software/
# 重命名
mv /usr/local/software/apache-zookeeper-3.8.4-bin/ /usr/local/software/zookeeper
```

配置环境变量

```bash
vim /etc/profile.d/my_env.sh

# 添加以下内容
export ZOOKEEPER_HOME=/usr/local/software/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

​	使用 `source /etc/profile.d/my_env.sh` 激活

创建 快照数据和日志 存储需要的文件夹

```bash
mkdir -p /home/zookeeper/data
mkdir -p /home/zookeeper/datalog
```

创建配置文件

```bash
vim /usr/local/software/zookeeper/conf/zoo.cfg

# 添加以下内容:

# 心跳单位，2s
tickTime=2000
# zookeeper-3初始化的同步超时时间，10个心跳单位，也即20s
initLimit=10
# 普通同步：发送一个请求并得到响应的超时时间，5个心跳单位也即10s
syncLimit=5
# 内存快照数据的存储位置
dataDir=/home/zookeeper/data
# 事务日志的存储位置
dataLogDir=/home/zookeeper/datalog
# 当前zookeeper-3节点的端口 
clientPort=2181
# 单个客户端到集群中单个节点的并发连接数，通过ip判断是否同一个客户端，默认60
maxClientCnxns=1000
# 保留7个内存快照文件在dataDir中，默认保留3个
autopurge.snapRetainCount=7
# 清除快照的定时任务，默认1小时，如果设置为0，标识关闭清除任务
autopurge.purgeInterval=1
#允许客户端连接设置的最小超时时间，默认2个心跳单位
minSessionTimeout=4000
#允许客户端连接设置的最大超时时间，默认是20个心跳单位，也即40s，
maxSessionTimeout=300000
#zookeeper-3 3.5.5启动默认会把AdminService服务启动，这个服务默认是8080端口
admin.serverPort=9001
#集群地址配置
server.1=hadoop101:2888:3888
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
```



## 6 安装Hadoop

Hadoop下载地址 [Apache Hadoop](https://hadoop.apache.org/releases.html)
推荐下载主流的3.3.x版本 选择Binary download - binary 进行下载 (文档书写时为[3.3.6](https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz))

上传至/opt后 ->

```bash
# 解压
tar -zxvf /opt/hadoop-3.3.6.tar.gz -C /usr/local/software/
# 重命名
mv /usr/local/software/hadoop-3.3.6/ /usr/local/software/hadoop
```

配置环境变量

```bash
vim /etc/profile.d/my_env.sh

# 添加以下内容: (root 为使用集群的 系统用户)

export HDFS_NAMENODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_ZKFC_USER=root
export HDFS_JOURNALNODE_USER=root
export HADOOP_SHELL_EXECNAME=root

export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root

export HADOOP_HOME=/usr/local/software/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop

export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

​	使用 `source /etc/profile.d/my_env.sh` 激活



## 7 编辑Hadoop配置文件

编辑/usr/local/software/hadoop/etc/hadoop/ 中的配置文件
	hadoopCluster 为集群的名字(可自由更改)

**hadoop-env.sh**
	`vim /usr/local/software/hadoop/etc/hadoop/hadoop-env.sh`
追加 ↓

```
export JAVA_HOME=/usr/local/software/jdk

export HDFS_NAMENODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_ZKFC_USER=root
export HDFS_JOURNALNODE_USER=root
export HADOOP_SHELL_EXECNAME=root

export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

**core-site.xml**
	`vim /usr/local/software/hadoop/etc/hadoop/core-site.xml`
将\<configuration\>标签替换为↓

```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoopCluster</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/hadoop/data</value>
  </property>
  <property>
    <name>ha.zookeeper.quorum</name>
    <value>hadoop101:2181,hadoop102:2181,hadoop103:2181</value>
  </property>
  <property>
    <name>hadoop.http.staticuser.user</name>
    <value>root</value>
  </property>
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>
  <property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
  </property>
  <property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
  </property>
</configuration>
```

**hdfs-site.xml**
	`vim /usr/local/software/hadoop/etc/hadoop/hdfs-site.xml`
将\<configuration\>标签替换为↓
	注：倒数第4个  \<property\> 标签，如果使用新建用户 需要替换为实际的系统用户昵称

```
<configuration>
  <property>
    <name>dfs.nameservices</name>
    <value>hadoopCluster</value>
  </property>
  
  <!-- HA模式有两个NameNode，分别是nn1，nn2 -->
  <property>
    <name>dfs.ha.namenodes.hadoopCluster</name>
    <value>nn1,nn2</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.hadoopCluster.nn1</name>
    <value>hadoop101:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.hadoopCluster.nn2</name>
    <value>hadoop102:8020</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.hadoopCluster.nn1</name>
    <value>hadoop101:9870</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.hadoopCluster.nn2</name>
    <value>hadoop102:9870</value>
  </property>
  
 <!-- 指定NameNode的edits元数据在JournalNode上的存放位置 -->
  <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://hadoop101:8485;hadoop102:8485;hadoop103:8485/hadoopCluster</value>
  </property>
  
  <!-- 指定该集群出故障时，哪个实现类负责执行故障切换 -->
  <property>
    <name>dfs.client.failover.proxy.provider.hadoopCluster</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  
 <!-- 配置隔离机制方法-->
  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
  </property>
  
  <!-- 使用sshfence隔离机制时需要ssh免登陆 -->
  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/root/.ssh/id_rsa</value>
  </property>
  
  <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/home/hadoop/journalnode/data</value>
  </property>
  
  <!-- 开启NameNode失败自动切换 -->
  <property>
    <name>dfs.ha.automatic-failover.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.safemode.threshold.pct</name>
    <value>1</value>
  </property>
</configuration>
```

workers
	`vim /usr/local/software/hadoop/etc/hadoop/workers`
清空内容并替换为↓

```
hadoop101
hadoop102
hadoop103
```

**mapred-site.xml**
	`vim /usr/local/software/hadoop/etc/hadoop/mapred-site.xml`
将\<configuration\>标签替换为↓

```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  
  <!-- yarn历史服务端口 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop101:10020</value>
  </property>
  
  <!-- yarn历史服务web访问端口 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop101:19888</value>
  </property>
</configuration>
```

**yarn-site.xml**
	`vim /usr/local/software/hadoop/etc/hadoop/yarn-site.xml`
将\<configuration\>标签替换为↓

```
<configuration>
    <!-- 启用resourcemanager -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    
    <!-- 集群ID设置的名字-->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>hadoopCluster</value>
    </property>
    
    <!--resourcemanager设置了两台互为备份-->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    
    <!--rm1与rm2的地址-->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop101</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>hadoop102</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>hadoop101:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>hadoop102:8088</value>
    </property>
    <!--配置zookeeper的地址-->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop101:2181,hadoop102:2181,hadoop103:2181</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    <!-- 是否将对容器实施物理内存限制 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
 
    <!-- 是否将对容器实施虚拟内存限制。 -->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
    <!-- 开启日志聚集 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
 
    <!-- 设置yarn历史服务器地址 -->
    <property>
        <name>yarn.log.server.url</name>
        <value>http://hadoop101:19888/jobhistory/logs</value>
    </property>
 
    <!-- 保存的时间7天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
</configuration>
```



## 8 克隆虚拟机

此时可以关闭虚拟机，保存快照，**完整克隆**2次虚拟机 分别为hadoop102、hadoop103

克隆完毕后分别为两个新建的虚拟机修改 Hostname 和 IP

```bash
# 修改主机名 替换NewName
hostnamectl set-hostname NewName

# 修改ip (ifcfg-ens33 替换为实际的文件名 一般为ens33)
vim /etc/sysconfig/network-scripts/ifcfg-ens33
	# 修改 IPADDR 条目
IPADDR=192.168.170.10x

# 重启网络
systemctl restart network
# 重启电脑
reboot
```



## 9 配置免密登录

**hadoop101**

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa

# 复制密钥
ssh-copy-id root@hadoop101
ssh-copy-id root@hadoop102
ssh-copy-id root@hadoop103

# 测试连接
ssh root@hadoop102
ssh root@hadoop103

# 分发密钥
scp -r ~/.ssh/ root@hadoop102:~/
scp -r ~/.ssh/ root@hadoop103:~/
```



## 10 配置zookeeper myid

**hadoop101**

```bash
echo 1 > /home/zookeeper/data/myid
```

**hadoop102**

```bash
echo 2 > /home/zookeeper/data/myid
```

**hadoop103**

```bash
echo 3 > /home/zookeeper/data/myid
```



## 11 Hadoop初始化

①启动zookeeper服务
**hadoop101、hadoop102、hadoop103**

```bash
zkServer.sh start
```

②启动JournalNode
	**hadoop101、hadoop102、hadoop103**

```bash
hdfs --daemon start journalnode
```

③初始化hdfs
**hadoop101** (选择有namenode的一台机器即可)

```bash
hdfs namenode -format
```

④将初始化后的元数据拷贝到另一个namenode节点 -> hadoop102, 目录是core-site.xml中hadoop.tmp.dir配置的数据源
**hadoop101**

```bash
scp -r /home/hadoop/data root@hadoop102:/home/hadoop
```

⑤启动namenode
**hadoop101**

```bash
hdfs --daemon start namenode
```

⑥在没有格式化的namenode服务器执行 -> 
**hadoop102**

```bash
hdfs namenode -bootstrapStandby
```

⑦启动第二个namenode
**hadoop102**

```bash
hdfs --daemon start namenode
```

⑧将namenode信息注册到zookeeper, 在任意以爱namenode节点执行->
**hadoop101**

```bash
hdfs zkfc -formatZK
```

⑨结束全部进程 ->
**hadoop101**

```bash
stop-dfs.sh
```

到此，Hadoop HA 集群就已经部署完毕了！



## 12 开启与关闭集群

### 开启

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
```

启动历史服务器(应该在配置好历史服务器的服务器进行启动) -> jobhistory服务
**hadoop101**

```bash
mapred --daemon start historyserver
```

此时 hadoop101 执行jps 应该出现以下进程：

```
16674 QuorumPeerMain
47330 JobHistoryServer
43347 ResourceManager
43556 NodeManager
47928 Jps
42842 DFSZKFailoverController
40635 JournalNode
42315 DataNode
42109 NameNode
```

### 关闭

**hadoop101**

```bash
mapred --daemon stop historyserver
stop-all.sh
```

**hadoop101、hadoop102、hadoop103**

```bash
zkServer.sh stop
```





## x 参考文献

[Hadoop3.x完全分布式详细配置_hadoop3部署-CSDN博客](https://blog.csdn.net/majingbobo/article/details/134357726?spm=1001.2014.3001.5502)

[Hadoop高可用集群搭建（超详细）_docker搭建hadoop高可用集群-CSDN博客](https://blog.csdn.net/weixin_46376562/article/details/110353143)