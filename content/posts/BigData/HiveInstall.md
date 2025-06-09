---
title: 'Hive 安装与配置'
date: 2024-06-22T22:41:20+08:00
draft: false
tags: ['BigData','Hadoop','Hive']
---

# HIVE



## 1 下载安装 Hive

Hive 下载官网：[Downloads](https://hive.apache.org/general/downloads/)
	下载地址 [Apache Download Mirrors](https://www.apache.org/dyn/closer.cgi/hive/) 文档演示[4.0.1版本](https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz)

上传至/opt目录后

```bash
# 解压
tar -zxvf /opt/apache-hive-4.0.1-bin.tar.gz -C /usr/local/software/
# 重命名
mv /usr/local/software/apache-hive-4.0.1-bin/ /usr/local/software/hive
```

配置环境变量

```bash
vim /etc/profile.d/my_env.sh

# 添加 ↓
export HIVE_HOME=/usr/local/software/hive
export HCATALOG_HOME=/usr/local/software/hive/hcatalog
export PATH=$PATH:$HIVE_HOME/bin:$HCATALOG_HOME/sbin
```

使用 `source /etc/profile.d/my_env.sh` 命令激活



## 2 配置 Hive

- 修改hive元数据存储信息路径、制定mysql数据库存储
  - hive-site.xml文件

进入路径 /usr/local/software/**hive/conf**
复制模板文件 `cp hive-default.xml.template hive-site.xml`

编辑 **hive-site.xml** 
在\<configuration\>标签添加下面内容 (使用 shift+G 快捷键快速到达文档低端)
	配置文件以 主机hadoop101 为例

```xml
    <!-- 记录HIve中的元数据信息  记录在mysql中 -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop101:3306/hive?useUnicode=true&amp;createDatabaseIfNotExist=true&amp;characterEncoding=UTF8&amp;useSSL=false&amp;serverTimeZone=GMT</value>
    </property>
 
    <!-- jdbc mysql驱动 -->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>
 
    <!-- mysql的用户名和密码 -->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    <!-- 日志目录 -->
    <property>
        <name>hive.querylog.location</name>
        <value>/user/hive/log</value>
    </property>
 
    <!-- 设置metastore的节点信息 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop101:9083</value>
    </property>
 
    <!-- 客户端远程连接的端口 -->
    <property> 
        <name>hive.server2.thrift.port</name> 
        <value>10000</value>
    </property>
    <property> 
        <name>hive.server2.thrift.bind.host</name> 
        <value>0.0.0.0</value>
    </property>
    <property>
        <name>hive.server2.webui.host</name>
        <value>0.0.0.0</value>
    </property>
 
    <!-- hive服务的页面的端口 -->
    <property>
        <name>hive.server2.webui.port</name>
        <value>10002</value>
    </property>
```

将 mysql数据库驱动上传至hive的lib目录

- mysql-connector-j-8.0.33.jar

删除lib目录中不匹配的jar包↓

- protobuf-java-3.23.4.jar

上传更新的jar包

- protobuf-java-4.28.3.jar

删除 hadoop与hive中版本冲突的 guava

```bash
# 删除hive中的guava后
rm -rf /usr/local/software/hive/lib/guava-22.0.jar
# 将hadoop中的guava拷贝到hive lib目录
cp /usr/local/software/hadoop/share/hadoop/common/lib/guava-27.0-jre.jar /usr/local/software/hive/lib/
```

修改**hive-site.xml**配置文件
	修改 hive.exec.local.scratchdir 、 hive.downloader.resources.dir
查找到位置后 修改为 ↓

```xml
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>/usr/local/software/hive/local</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/usr/local/software/hive/resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
```

## 3 初始化 Hive

执行命令

```bash
schematool -initSchema -dbType mysql -verbose
```

执行完成后 出现 "Initialization script completed"，此时mysql中就已被自动创建了 hive 数据库

## 4 启动 Hive

方式1 - 前台启动

```bash
hive --service metastore
hive --service hiveserver2
```

方式2 - 后台启动

```bash
hive --service metastore &
hive --service hiveserver2 &
```

远程连接：

```bash
beeline -u jdbc:hive2://IP地址:10000 -n root
```

