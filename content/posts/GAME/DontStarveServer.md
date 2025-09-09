---
title: '饥荒联机版服务器搭建教程(Ubuntu22)'
date: 2024-06-25T22:45:11+08:00
draft: false
tags: ['GAME']
---

# Don't Starve Together

## MAC系统开服教程

### 1 开启在线服务器

1️⃣ 饥荒联机版游戏启动并登入后, 点击 [账户], 会跳转到网页

2️⃣ 保存[Klei用户ID] (KU_XXXXXXX)

3️⃣ 点击菜单栏的 [游戏] [添加新服务器] [配置服务器]

``` 
// 保存服务器票据
pds-xxxxxxxxxxxxxxxxxxxxxxxxxxx
```

4️⃣ 创建存档 设置MOD

```
// Mod对应 id
458587300 - 攻击范围显示
375850593 - 护身符和背包栏
378160973 - 全球定位
375859599 - 血量显示
2287303119 - Show Me
458587300 - 木牌传送
```

5️⃣ Steam下载开服工具 [Don't Starve Together Dedicated Server]

右键浏览 开服工具 本地文件 -> 进入App包内容 -> 进去Contents文件夹 -> 右键[MacOS] 和 [mods] 两个文件夹, 制作替身 -> 将两个快捷方式放到桌面

6️⃣ 右键浏览 饥荒联机版 本地文件 -> 将Contents下mods文件夹内的所有内容 拷贝 到开服工具的mods文件夹中 选择[替换]

7️⃣ 右键MacOS替身 新建终端 在终端输入>>>

```bash
# 建立主服务器脚本
echo "./dontstarve_dedicated_server_nullrenderer -console -cluster Cluster_1 -shard Master" > master_start.sh
# 建立洞穴服务器脚本
echo "./dontstarve_dedicated_server_nullrenderer -console -cluster Cluster_1 -shard Caves" > cave_start.sh
# 设置脚本权限
chmod +x master_start.sh cave_start.sh
```

8️⃣ 启动服务器

```bash
./master_start.sh
./cave_start.sh

# 关闭服务器只需要Ctrl+C
```

注：可能会出现 Don't Starve Together Dedicated Server.app 已损坏的报错，需要进入 [系统设置] -> [隐私与安全性] -> 最下方[安全性] 允许开服工具的打开

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
