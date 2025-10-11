---
title: '饥荒联机版服务器搭建教程(Mac || Ubuntu22)'
date: 2024-06-25T22:45:11+08:00
draft: false
tags: ['GAME']
---

# Don't Starve Together

Last Update Time - 2025/10/06 - 02:33

## MAC

1️⃣ 饥荒联机版游戏启动并登入后, 点击左下角 [账户], 会跳转到网页,**保存[Klei用户ID]** (KU_XXXXXXX); 

2️⃣ 点击网页菜单栏的 [游戏] [添加新服务器] [配置服务器]

```
// 保存服务器票据
pds-xxxxxxxxxxxxxxxxxxxxxxxxxxx
```

3️⃣ 在饥荒联机版游戏中**创建存档**, 设置Mod. 将存档文件(/User/yyz/Documents/Klei/DoNotStarveTogether/xxx数字idxxx/Cluster_1)**拷贝到**他的**上一层级目录**(/User/yyz/Documents/Klei/DoNotStarveTogether)

<!--后续创建的开服脚本会自动寻找"/User/yyz/Documents/Klei/DoNotStarveTogether/Cluster_1"目录-->

4️⃣ 在刚才拷贝的存档文件夹中创建文件 **cluster_token.txt**, 并在其中填入服务器票据pds-xxxxx

5️⃣ 在刚才拷贝的存档文件夹中创建文件 **adminlist.txt**, 并在其中填入Klei用户id

<!--可以通过MAC自带的[文本编辑]软件创建txt文件编写,填写完成后 点击菜单栏的格式,制作纯文本,保存即可.-->

6️⃣ Steam下载开服工具 [Don't Starve Together Dedicated Server]

右键浏览 开服工具 本地文件 -> 进入App包内容 -> 进去Contents文件夹 -> 右键[MacOS]文件夹, 制作替身 -> 将快捷方式放到桌面

右键[MacOS]文件夹 -> 新建终端 -> 在终端输入>>>

```bash
# 建立主服务器脚本
echo "./dontstarve_dedicated_server_nullrenderer -console -cluster Cluster_1 -shard Master" > master_start.sh
# 建立洞穴服务器脚本
echo "./dontstarve_dedicated_server_nullrenderer -console -cluster Cluster_1 -shard Caves" > cave_start.sh
# 设置脚本权限
chmod +x master_start.sh cave_start.sh
```

7️⃣ **设置Mod**. 进入开服工具目录 -> 进入Contents文件夹

```
// 将下面追加书写到服务端文件目录下的 mods/dedicated_server_mods_setup.lua中实现自动安装更新mod
ServerModSetup("378160973")
ServerModSetup("362175979")
ServerModSetup("2477889104")
ServerModSetup("2823530744")
ServerModSetup("2189004162")
ServerModSetup("1207269058")
ServerModSetup("1185229307")
ServerModSetup("2119742489")
ServerModSetup("2505341606")
ServerModSetup("1595631294")
```

8️⃣ **开服** 右键[MacOS]的替身 -> 新建终端 -> 在终端输入>>>

```bash
./master_start.sh
./cave_start.sh

# 关闭服务器只需要Ctrl+C
```

**注：**可能会出现 Don't Starve Together Dedicated Server.app 已损坏的报错，需要进入 [系统设置] -> [隐私与安全性] -> 最下方[安全性] 允许开服工具的打开

参考教程 : https://www.bilibili.com/video/BV1o34y1s7A6



## Ubuntu 

华为云 - Ubuntu22.04 - 2核4G 5M宽带

### 1 更新软件 安装依赖

```bash
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386

sudo apt update
sudo apt upgrade
sudo apt install libstdc++6 libgcc1 libcurl4-gnutls-dev:i386 lib32z1 lib32stdc++6

```

### 2 安装SteamCMD和饥荒联机版

```bash
mkdir ~/steamcmd
cd ~/steamcmd

wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
./steamcmd.sh
#输入左侧出现“steam>”说明steamcmd启动成功

force_install_dir "../dontstarvetogether_dedicated_server"
login anonymous
app_update 343050 validate
#当执行完成后输入'quit'退出steam服务
quit
```

### 3 复制存档和世界数据

**游戏物理机**：

在**游戏中**创建世界,勾选Mod.

在**新建存档**目录下的**Cluster_1**文件夹中创建两个文件

- **cluster_token.txt** - 放置**服务器令牌**
- **adminlist.txt** - 放置**管理员Klei id**

最后在**cluster.ini**文件中修改昵称、密码等数据

### 4 搬运创建好的数据

```bash
#在服务器中创建存档的文件夹
mkdir -p ~/.klei/DoNotStarveTogether/Cluster_1

#创建启动脚本
cd ~
nano boot.sh

>>>
#!/bin/bash

steamcmd_dir="$HOME/steamcmd"
install_dir="$HOME/dontstarvetogether_dedicated_server"
cluster_name="Cluster_1"
dontstarve_dir="$HOME/.klei/DoNotStarveTogether"

function fail() {
    echo Error: "$@" >&2
    exit 1
}

function check_for_file() {
    if [ ! -e "$1" ]; then
        fail "Missing file: $1"
    fi
}

cd "$steamcmd_dir" || fail "Missing $steamcmd_dir directory!"
check_for_file "steamcmd.sh"
check_for_file "$dontstarve_dir/$cluster_name/cluster.ini"
check_for_file "$dontstarve_dir/$cluster_name/cluster_token.txt"
check_for_file "$dontstarve_dir/$cluster_name/Master/server.ini"
check_for_file "$dontstarve_dir/$cluster_name/Caves/server.ini"
check_for_file "$install_dir/bin"
cd "$install_dir/bin64" || fail
run_shared=(./dontstarve_dedicated_server_nullrenderer_x64)
run_shared+=(-console)
run_shared+=(-cluster "$cluster_name")
run_shared+=(-monitor_parent_process $$)
run_shared+=(-shard)
"${run_shared[@]}" Caves | sed 's/^/Caves: /' &
"${run_shared[@]}" Master | sed 's/^/Master: /'

#设置权限
sudo chmod u+x boot.sh
```

### 5 添加mod

在/root/dontstarvetogether_dedicated_server/mods下的**dedicated_server_mods_setup.lua**添加如下类型的数据

```
ServerModSetup("2321974509")
ServerModSetup("362175979")
ServerModSetup("375850593")
ServerModSetup("378160973")
ServerModSetup("537902048")
ServerModSetup("666155465")
```

即可实现**自动更新Mod**

### 6 开放端口

UDP协议开放10888、10998、10999

TCP和UDP协议开放8767、27017

### 7 启动服务器

```bash
#启动服务器
cd ~
nohup ./boot.sh>root.log 2>&1 &

#关闭服务器
ps -ef | grep don
kill 服务器pid

#执行后饥荒服务器会在后台运行，可以通过下面这个命令查看输出的日志
tail -f root.log

#服务器性能查看
top
```



## MOD List

```
// Mod对应 id
378160973 - 全球定位
362175979 - 虫洞标记
2477889104 - 驯牛信息
2823530744 - 背包栏位
2189004162 - Insight (ShowMe+)
1207269058 - 简易血条
1185229307 - 史诗血量条
2119742489 - 木牌传送
2505341606 - 防卡两招
1595631294 - 箱子小木牌
```

