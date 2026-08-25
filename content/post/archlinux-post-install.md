+++
date = '2024-10-25T23:23:00+08:00'
updated = 2026-08-25T10:30:00+08:00
draft = false
title = 'Archlinux完善指南'
slug = 'archlinux-post-install'

[taxonomies]
categories = ['wiki']
tags = ['linux', 'archlinux', 'system']
+++

Archlinux安装完成后的一些日常使用提升体验的操作

<!--more-->


# [**"时刻关注社区最新动态，一切以官方文档为准"**](https://www.archlinuxcn.org/)

## 系统管理

普通用户创建(可选，但建议创建)

> **用户**：应当仅在需要系统管理时使用root

使用useradd添加用户

-   -m, 创建用户主目录，即 /home/username
-   -G, 将用户加入附加组，通常加入到 wheel 组，可以使用 sudo 和 su 命令权限管理
-   -s, 指定用户默认登录的 shell 的路径，通常应该使用已经正确配置在 /etc/shells 中

```bash
#使用查看可以使用的shell，默认是bash
chsh -l
#添加用户,我的username是otoya
useradd -m -G wheel -s /bin/bash otoya
#为该用户设置密码
passwd otoya

#修改 /etc/sudoers 文件来使 wheel 组的用户可执行任何命令
#为确保 /etc/sudoers 文件格式完全正确，因此使用 visudo 来编辑该文件防止出错，visudo 默认调用 vi 作为编辑器
#也可以临时调用其他编辑器，只需在该命令前加上 EDITOR 环境变量即可，本例使用 Vim
EDITOR=vim visudo
#取消注释该条目，以允许 wheel 组的所有用户以任何用户(包括root)的身份执行任何命令
%wheel ALL=(ALL:ALL) ALL
```

系统服务管理 systemctl基本用法

```bash
status查看状态
start立即启动
stop立即停止
restart立即重启
enable开机时启用
disable取消开机时启动
enable --now启用并立即启动
```

启用NetworkManager

```bash
systemctl enable --now NetworkManager
```

然后联网，要么用nmcli要么iwd

```bash
#用nmcli联网
nmcli
nmcli device wifi list
nmcli device wifi connect SSID password PASSWORD
#nmcli的密码会明文显示好家伙


#用iwd联网(可选)
#先启动
systemctl enable --now iwd

#进入 iwctl 交互提示符
iwctl

#列出所有wifi设备
device list
#输出: 一般为 wlan0，本次以wlan0为例

#扫描网络
station wlan0 scan

#列出所有可用网络
station wlan0 get-networks

#连接网络(不支持中文wifi名)
station wlan0 connect SSID
#输入密码
#wifi密码并不会显示，保证输入正确回车即可
```


测试网络连接

```bash
ping bing.com
```

重启

```bash
reboot
```

## 软件包管理

启用 Arch linux 中文社区仓库,依然采用 USTC 源(可选)
编辑 /etc/pacman.conf
```bash
vim /etc/pacman.conf
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```
刷新数据库并更新系统

```bash
sudo pacman -Syyu
```
```bash
#添加中文社区仓库后，必须先安装 archlinuxcn-keyring 钥匙环，才能安装其他软件，否则会出现 gpg 错误
# 而 archlinuxcn-keyring 这个包是用 farseerfc 老哥的 key 签署验证的，本来 Arch Linux 的官方 keyring 中也包含了他的密钥但是去年12更新中删除了一个主密钥，导致farseerfc老哥密钥信任不足，所以要手动进行信任
# sudo pacman-key --lsign-key "farseerfc@archlinux.org" 
#信任后再安装 archlinuxcn-keyring 钥匙环 
#现在已经不需要手动信任了，直接安装就行

sudo pacman -S archlinuxcn-keyring
```

如果想使用32位程序，推荐启用 multilib 库，可在64位系统上运行和构建32位程序(可选)

```bash
#编辑 /etc/pacman.conf
#取消注释 multilib 两行
[multilib]
Include = /etc/pacman.d/mirrorlist
```

也可以添加其他第三方库，[链接](https://wiki.archlinuxcn.org/wiki/%E9%9D%9E%E5%AE%98%E6%96%B9%E7%94%A8%E6%88%B7%E4%BB%93%E5%BA%93)


当然，也可以通过安装 yay 或者 paru 来使用 aur 仓库(可选)
```bash
sudo pacman -S yay
```


## 显卡驱动与视频硬解
咱是amd核显，所以需要安装amd显卡驱动

```bash
#mesa提供基础的3d加速和视频硬解，vulkan为高性能的图形渲染和计算提供支持
#lib32则是为32位程序提供了支持
sudo pacman -S mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon
```

## 音视频系统与蓝牙
pipewire是个新的底层多媒体框架，而且兼容旧的音频框架
```bash
sudo pacman -S pipewire wireplumber pipewire-audio pipewire-alsa pipewire-pulse pipewire-jack
#如果想要支持32位系统，就在包前加上lib32，例如lib32-pipewire-pulse
#另外，安装了pipewire-pulse后，pipewire会自动处理蓝牙
#至于wireplumber，他是个会话管理器，跟路由器差不多
#然后启用
#因为pipewire使用systemd/用户管理服务器并自动激活socket，所以不需要sudo
systemctl --user enable --now pipewire wireplumber
```
蓝牙配置
首先安装相关的包
```bash
sudo pacman -S bluez bluez-utils
#然后启用
sudo systemctl enable --now bluetooth
#然后你就可以吧连牙蓝上了
```

## 快照管理
关于快照管理其实有好几种方案，像timeshift、snapper，还有Yabsnap。
就先以snapper为例吧，剩下的以后再补全好了。
我们先来安装snapper和btrfs的子卷可视化工具btrfs-assistant(本来不是很想用的，但是真香.wav)
```bash
sudo pacman -S snapper btrfs-assistant
```
然后创建配置
```bash
sudo snapper -c root create-config /
```
这会生成/.snapshots子卷，我寻思这真的是子卷吗，怎么看都象是个普通的文件夹
因为不能umount但是可以直接删除，所以我也搞不清楚什么情况
不过wiki里说是那就算是吧
然后我们需要删除这个子卷
```bash
sudo rm -rf /.snapshots
```
因为之前我们创建过@snapshots子卷，但是我们没有挂载，现在是时时候挂载了
首先创建一个挂载点/.snapshots , 对，你没有看错，不过，这次它的身份是挂载点而不是子卷
```bash
mkdir /.snapshots
```
然后挂载@snapshots子卷到/.snapshots
```bash
mount -o noatime,compress=zstd,subvol=@snapshots /dev/nvme0n1p2 /.snapshots
```
然后编辑fstab挂载信息
```bash
sudo vim /etc/fstab
```
在fstab中添加以下行(参数不影响)
```bash
#/dev/nvme0n1p2
UUID=根分区UUID /.snapshots btrfs noatime,compress=zstd,subvol=@snapshots 0 0
```
如何查看uuid
```bash
blkid
```
或者vim打开fstab，直接yy复制除了esp挂载信息的其他行然后p黏贴，然后改一改就行了。


复查
```bash
df -h
lsblk -f
findmnt -t btrfs
cat /etc/fstab
sudo btrfs subvolume list /
#其中，如果有如下输出，则说明子卷相互独立
ID 256 gen 75 top level 5 path @
ID 257 gen 67 top level 5 path @home
ID 258 gen 14 top level 5 path @swap
ID 259 gen 10 top level 5 path @snapshots
ID 260 gen 78 top level 5 path @var_log
ID 261 gen 19 top level 256 path var/lib/portables
ID 262 gen 19 top level 256 path var/lib/machines
#可以看到，@ @home @swap @snapshots @var_log都是平级，都是在top level5底下的平级子卷
```


然后重启系统


接着首先查看.snapshots是否为空文件夹
它应当为空
```bash
ls /.snapshots
```
接着你可以测试创建一个根分区的快照，
```bash
sudo snapper -c root create -d "testTESTbefore"
```
然后查看是否正确创建了,并记下快照id

再次复查.snapshots文件夹
它应当至少包含一个文件夹，文件夹名就是快照id
```bash
ls /.snapshots
```
例如他可能回会输出
```bash
$ ls /.snapshots/
1
$ ls /.snapshots/1/
info.xml  snapshot
```



确认无误后，创建一个文件做参考


```bash
sudo snapper -c root list
```
确认无误后，创建一个文件做参考
```bash
vim test.txt
```
然后随便写入文本

接着用btrfs-assistant查看快照，并且对比快照id
```bash
sudo btrfs-assistant -l
```
然后 Let's rock and roll!
```bash
sudo btrfs-assistant -r 快照id
```
接着重启
```bash
reboot
```
然后发现，系统启动了，但是test.txt文件不见了
测试成功



然后我们可以开启grub-btrfs服务，生成grub快照菜单
```bash
sudo systemctl enable --now grub-btrfsd
```
然后生成grub配置文件
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
然后重启系统，即可在grub菜单中看到快照选项了

另外，你可以通过cron守护进程定时创建快照，也可以通过snapper提供的两个脚本来定时创建和删除快照。
```bash
#自动按时创建快照
snapper-timeline.timer
#定期清理老旧快照
snapper-cleanup.timer
```
当中，个人并没有启用定时创建快照，因为个人使用，空间本来就不是很大，只开启定时清理老旧快照
可以通过
```bash
sudo systemctl enable --now snapper-cleanup.timer
```
来开启定时清理，然后来来进行具体配置
编辑配置文件
```bash
sudo vim /etc/snapper/configs/root
```
禁止自动创建快照(虽然没开启服务，但是双重保险)，自动清理
```bash
TIMELINE_CREATE="no"
TIMELINE_CLEANUP="yes"
```
配置最多保留快照数，以及最多重要快照数
```bash
NUMBER_LIMIT="10"
NUMBER_LIMIT_IMPORTANT="5"
```

当然也可以通过
```bash
sudo snapper -c root list
```
来查看root上的已有的分区，如果想要手动分区，则根据快照id(例如某个id为1)，通过
```bash
sudo snapper -c root delete 1
```
来删除，但是删除后之后的快照并不会延续1的位置，而是空出来，直接显示为2,不过无关大雅

然后可以通过安装 snap-pac 包，自动在使用 pacman 更新系统或者安装/卸载软件时自动调用 snapper 创建快照
```bash
sudo pacman -S snap-pac
```

其实到这里基本就已经解决了大部分问题了，本篇完。