---
title: 'Arch Install - 在物理机安装ArchLinux的步骤'
date: 2024-05-28T11:09:46+08:00
draft: false
tags: ['Linux','ArchLinux']
cover:
    image: img/head.jpg # image path/url
    alt: 'This is a ArchLinux Logo' # alt text
    caption: 'ArchLinux' # display caption under cover
    # relative: false # when using page bundles set this to true
    # hidden: true # only hide on current single page
---
> Author - yyz
>
> Create Time - 2024/5/xx
>
> **Last Update Time - 2024/5/23**

# 0 Basic Settings

- BIOS设置中 **Secure Boot** 设置为 **Disable**
- (双系统) Win11电源管理计划 **关闭快速启动**

# 1 Arch Install

Win11 & Archlinux

- Btrfs文件系统

## 1.1 TTY CMD:

```bash
#禁用reflector服务
systemctl stop reflector.service
#确认为UEFI模式 - 出东西就是UEFI模式
ls /sys/firmware/efi/efivars

#无线连接网络 - 有线可以不用设置
iwctl # 进入交互式命令行
device list # 列出无线网卡设备名，比如无线网卡看到叫 wlan0
station wlan0 scan # 扫描网络
station wlan0 get-networks # 列出所有 wifi 网络
station wlan0 connect wifi-name # 进行连接
exit # 连接成功后退出

#更新时间
timedatectl set-ntp true
timedatectl status

#配置镜像源
nano /etc/pacman.d/mirrlist
#第一行添加 Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch

#查看分区列表
fdisk -l
#编辑分区
cfdisk [/dev/nvme?]

#---
#分区方案
# /dev/nvme1n1p1 - SWAP 16G
# /dev/nvme1n1p2 - LinuxFile 420G
# /dev/nvme0n1p1 - EFI 1G - Win11引导文件,不用格式化直接挂载
#---

#格式化分区
#mkfs.fat -F32 /dev/nvmexxxx #格式化EFI分区
mkfs.btrfs -f -L ArchOS /dev/nvme1n1p2
mkswap -L SWAP /dev/nvme1n1p1

#---
#子卷方案
#@ : /mnt
#@home : mnt/home
#---

#挂载Btrfs分区到/mnt
mount -t btrfs -o compress=zstd /dev/nvme1n1p2 /mnt
#创建子卷 - timeshift支持模式
btrfs subvolume create /mnt/@ # 创建 / 目录子卷
btrfs subvolume create /mnt/@home # 创建 /home 目录子卷
btrfs subvolume list -p /mnt #复查子卷情况
#卸载Btrfs分区
umount /mnt

#挂载根目录 *一定要先挂载根目录
mount -t btrfs -o subvol=/@,compress=zstd /dev/nvme1n1p2 /mnt
#挂载/home目录
mkdir /mnt/home
mount -t btrfs -o subvol=/@home,compress=zstd,ssd /dev/nvme1n1p2 /mnt/home
#挂载/boot目录
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
#挂载swap分区
swapon /dev/nvme1n1p1

#更新软件包
pacman -Syy
#重新安装密钥
pacman -S archlinux-keyring

#安装系统
pacstrap /mnt base base-devel\
linux-zen linux-zen-headers linux-firmware btrfs-progs\
grub os-prober efibootmgr ntfs-3g amd-ucode\
networkmanager sudo bluez bluez-utils\
nano
#---
#amd-ucode(AMD的微码,英特尔则为 intel-ucode)
#grub(引导系统)
#os-prober(多系统检测,双系统安装)
#efibootmgr(efi启动项管理)
#---

#创建fstab(自动挂载配置文件)
genfstab -U /mnt >> /mnt/etc/fstab

#使用chroot进入系统
arch-chroot /mnt
```

## 1.2 Chroot CMD

```bash
#设置主机名
nano /etc/hostname
#设置时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#设置硬件时间
hwclock --systohc

#(很可能不用)配置 /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   主机名.localdomain 主机名

#本地化
nano /etc/locale.gen
#-删除 en_US.UTF-8 和 zh_CN.UTF-8 前的 #
#生成locale
locale-gen
#添加语言
nano /etc/locale.conf
#-添加 LANG=en_US.UTF-8

#-直接填写即可
#设置root密码
passwd
#创建新用户
useradd -m -G wheel [username]
#设置用户密码
passwd [username]
#赋予用户root权限 - 编辑sudoers文件
nano /etc/sudoers
#-删除 %wheel ALL=(ALL:ALL) 前的 #

#启动服务
systemctl enable NetworkManager
systemctl enable bluetooth

#安装grub服务
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
#编辑grub
nano /etc/default/grub
#-删除 GRUB_DISABLE_OS_PROBER=FALSE 前的 #
#-GRUB_CMDLINE_LINUX_DEFAULT="loglevel=5 nowatchdog" #去掉quiet参数,加入nowatchgod,提升开机速度

#更新gurb引导
grub-mkconfig -o /boot/grub/grub.cfg

#刷新软件包
sudo pacman -Syy
#安装桌面环境 - KDE
pacman -S xorg plasma
#开机启动sddm服务 - 显示管理服务
systemctl enable sddm
#安装其他必要软件
pacman -S konsole dolphin ark kate
#---
#konsole(终端)
#dolphin(文件管理器)
#ark(解压缩工具)
#kate(文本编辑器)
#---
#安装其他软件 - 字体,输入法框架,git,firefox,gwenview(图片查看器)
pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra wqy-zenhei\
fcitx5-im fcitx5-qt fcitx5-gtk fcitx5-chinese-addons fcitx5-material-color\
fcitx5-pinyin-zhwiki fcitx5-pinyin-moegirl\
firefox git
#配置集成输入法 - /etc/environment
#添加这几行
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
INPUT_METHOD=fcitx
GLFW_IM_MODULE=ibus
#-
#退出系统并重启
exit
umount -R /mnt
reboot
```

## 1.3 Basic Setting

```bash
#重启后更新grub - 使grub找到windows启动项
grub-mkconfig -o /boot/grub/grub.cfg

#开启32位支持库 配置国内源
sudo nano /etc/pacman.conf
#-去掉 [multilib]及下一行前的 #
#-添加 [archlinuxcn] Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
#刷新软件包
sudo pacman -Syy
sudo pacman -Syyu
#导入密钥
sudo pacman-key --lsign-key "farseerfc@archlinux.org"
sudo pacman -S archlinuxcn-keyring

#AUR Helper - paru
pacman -S paru
```

## 1.4 Snapshots - 使用timeshift工具

```bash
sudo pacman -S timeshift # 安装Timeshift
sudo systemctl enable --now cronie.service # 启用Crond服务，启用该服务后Timeshift才能定期自动创建快照

#删除subvolib
sudo sed -i -E 's/(subvolid=[0-9]+,)|(,subvolid=[0-9]+)//g' /etc/fstab
#即删除/etc/fstab文件中 /和/home条目中最后的subvolid=xxx参数
```

> [!IMPORTANT]
>
> 完成后建议执行命令删除 **subvolib**,否则恢复BTRFS类型快照时无法挂载目录，导致**无法正常进入系统**。



# 9 Others

## 9.1 一些问题的解决方式

```bash
#archlinuxcn keyring安装失败解决方式
https://github.com/archlinuxcn/repo/issues/3557
```

## 9.2 解决机械革命蛟龙系列 键盘无法使用的问题

```bash
sudo pacman -S acpica cpio
#执行反编译
cat /sys/firmware/acpi/tables/DSDT > dsdt.dat
iasl -d dsdt.dat
#执行后生成dsl文件,进行编辑
#-修改PS2K设备下的IRQ(Edge, ActiveLow, Shared, ) 中的 ActiveLow 为 ActiveHigh
#-修改DefinitionBlock("","DSDT",2,....中最后的步进值 +1

#打包补丁并应用
iasl dsdt.dsl

mkdir -p kernel/firmware/acpi
cp dsdt.aml kernel/firmware/acpi
find kernel | cpio -H newc --create > acpi_override
cp acpi_override /boot/acpi_override
echo "GRUB_EARLY_INITRD_LINUX_CUSTOM=\"acpi_override\"" >>/etc/default/grub

#更新grub引导
grub-mkconfig -o /boot/grub/grub.cfg
```

## 9.3 备份软件列表方法 - 未测试

```bash
#生成软件包列表
pacman -Qqen > packages-repository.txt
pacman -Qqem > packages-AUR.txt

#重新安装
pacman --needed -S - < packages-repository.txt
cat packages-AUR.txt | paru -S --needed --noconfirm
```

