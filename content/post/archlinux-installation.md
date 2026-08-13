+++
date = '2024-03-11T01:01:40+08:00'
draft = false
title = 'Archlinux安装指南'
slug = 'archlinux-installation'

[taxonomies]
categories = ['wiki']
tags = ['linux', 'archlinux', 'system']
+++

Archlinux安装指南参考

<!--more-->

# **[“时刻关注社区最新动态，一切以官方文档为准”](https://www.archlinuxcn.org/)**

## 准备工作

> **注意**：默认已经从仓库下载好了最新的\*\*[安装镜像](https://mirrors.ustc.edu.cn/archlinux/iso/)\*\*，且写入或者存入u盘(推荐 Ventoy)，且已经重启进入 Archlinux live 环境

### 配置控制台字体(可选)

1. 列出可选字体
   ```bash
   ls /usr/share/kbd/consolefonts/
   ```
2. 设置字体(可自行选择合适字体)
   ```bash
   setfont ter-132b
   ```
3. 更改 wifi 名为英文，无线连接时不支持中文 SSID

## 网络设置

> **网络设置**：二选任意一个连接成功即可

### 网线直连

连接网线后开箱即用，ping通后即可继续后续步骤

```bash
ping baidu.com
```

### wifi连接

```bash
#查看wifi软、硬件是否被禁用
rfkill list
#如果被禁用,使用以下命令解除限制
rfkill unblock all
```

解除后使用 iwctl 连接网络

```bash
#进入 iwctl 交互提示符
iwctl

#列出所有wifi设备
device list
输出: 一般为 wlan0，本次以wlan0为例

#扫描网络
station wlan0 scan

#列出所有可用网络
station wlan0 get-networks

#连接网络(不支持中文wifi名)
station wlan0 connect SSID
#wifi密码并不会显示，保证输入正确回车即可
#退出iwd
exit
#连接网络，确保畅通
ping baidu.com
```

## ssh连接(可选)
最好换源先，不然openssh下载慢

```bash
#使用vim编辑 /etc/pacman.d/mirrorlist 
vim /etc/pacman.d/mirrorlist
#在文件的最顶端添加 USTC 镜像源以加速下载
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#刷新数据库
pacman -Sy
```

```bash
#设置root账户密码
passwd
#输入密码

#安装openssh
pacman -Sy openssh

#临时启动openssh服务
systemctl start sshd

#查看笔记本ip (ipv4地址)
ip a
#假设输出设备ip为 192.168.31.106

#在另一台设备上远程连接需要安装Archlinux的设备
ssh root@192.168.31.106
#接着输入密码

#成功远程连接,之后可以使用archinstall脚本或者跟随以下步骤进行后续安装操作。
#archinstall脚本真好用(真香.mp4),以前这个脚本确实算不上好用，但是这些年来一直在更新
#体验也确实越来越好，这下arch真就彻彻底底的编程新手/懒人发行版了(受虐滑稽.jpg
```

## 更新系统时间

```bash
timedatectl
```

## 分区

```text
关于esp分区的问题，首先esp分区目前来说必须是fat文件系统，问就是规范，希望以后能改革一下(不是。
关于挂载点，要么/boot，要么/efi，目前来说都是推荐/efi，说是方便全盘加密。
关于加密问题，首先esp分区肯定不能加密。因为如果加密了，不管是bios还是uefi都读取不到了。然后/boot下通常默认是放cpu微码文件和内核文件以及grub主题配置文件还有initramfs等文件的默认路径，/boot实际上是可以被加密的。但如果把esp分区也挂载到/boot下，当全盘加密时，加密工具会尝试将/boot分区也加密，但是因为esp分区无法被加密，这可能会导致系统无法启动，因为uefi读不到esp分区信息。但如果把esp分区挂载到/efi下，这样，你即便全盘加密，比如加密了/boot，也不会影响到/efi分区里的东西。

关于esp分区实际的挂载方式
挂载esp分区到/boot。此时需要将esp分区分配大一点的空间，因为此时/boot相当于一个独立的分区，它挂载到了系统根分区里的/boot文件夹，当访问/boot文件夹时，实际上时访问esp分区的内容。
挂载esp分区到/efi。此时，由于/boot是根分区下的一个普通文件夹，尽管它仍然是微码等文件的默认路径，但由于esp分区的文件会存入/efi文件夹，所以不需要给esp分区分配特别大的空间。
至于两种情况应该分配多大的空间才算合适，
个人经验来看，挂载到/boot的话，esp分区给2个GiB就已经非常充足了。
挂载到/efi下的话，1个GiB也非常充足了。
其实500MiB也完全没问题

关于home分区其实也是类似，可以做独立分区(传统的ext4文件系统)，也可以像根分区下的普通文件夹一样(比较现代的btrfs等文件系统)，不必单独分区。

关于分区大小
大部分分区工具其实都默认二进制，比如即使输入2G,或者输入2，也默认会写入2GiB
因而在分区时也需要按照二进制来进行分配。
比如512G的固态，换算成二进制也就是512x0.931=476GiB(近似)


关于swap分区/swap文件的问题，首先，他们不是必须的。还是看个人习惯。如果内存管够，而且用完就关机，那其实完全可以不要swap分区/文件。但如果需要休眠，或者内存比较小，那就需要它来做临时写入。说到底，他就是拿储存换内存的东西。
关于swap分区和文件的选择，首先，swap文件灵活性比较好，你甚至可以装完系统再创建它，调整也很方便。而swap分区的话，理论上稳定性更好，性能略高于swap文件，不过感受不是很强。不过对于桌面用户来说，也许swap文件是个更加方便好用的选择。不过看个人喜好选择，反正都能用。

关于睡眠(Suspend to RAM)和休眠(Suspend to Disk)以及混合睡眠(Hybrid Suspend)。首先睡眠的话，所有东西会保存在内存中，所以会保持内存通电，其余大部分硬件断电。因此不需要swap分区/文件参与其中。所以如果突然断电，之前的状态便不复存在。而休眠的话，会将内存中所有内容完整写入swap分区/文件，然后完全关机。毕竟写入到存储了，所以断电后也能重新完整读取。所以说，swap分区/文件大小还是大于或等于物理内存要稳妥一点，毕竟来都来了。
而至于这个混合睡眠，相当于是拿休眠给睡眠做了一个保险。它会先把内存写入swap分区/文件，然后保持内存供电。这样启动快，而且万一突然断电，重启后仍然能恢复之前的状态。所以它也需要swap分区/文件。


```

### 使用ext4文件系统
```text
(这个方案选择esp分区挂载到/boot，独立home分区，不分配swap分区/文件)
```
#### 创建分区

```bash
#查看当前硬盘分区情况
lsblk
#或者
fdisk -l

#输出可能为 sda 或者 nvme 硬盘，确定需要安装的硬盘，本文以 nvme 硬盘为例

#使用分区工具 cfdisk、fdisk、parted 等 修改分区表，以 cfdisk 为例
cfdisk /dev/nvme0n1 #本例以1t固态分区为例(财大气粗)
```

|       设备       |       分区类型       | 分区大小 |    挂载点    | 挂载顺序(格式化后挂载) |
| :------------: | :--------------: | :--: | :-------: | :----------: |
| /dev/nvme0n1p1 |    Efi System    |  2G  | /mnt/boot |       2      |
| /dev/nvme0n1p2 | Linux filesystem | 120G |    /mnt   |       1      |
| /dev/nvme0n1p3 |    Linux home    | 剩余空间 | /mnt/home |       3      |


> **注意**：如果像要swap分区，此处可以分配/dev/nvme0n1p4 | Linux swap | 物理内存大小或稍微大一点 | 最后挂载   


#### 格式化分区

```bash
#根分区和home分区使用ext4文件系统
mkfs.ext4 /dev/nvme0n1p2
mkfs.ext4 /dev/nvme0n1p3

#efi系统分区格式化为fat32
mkfs.fat -F 32 /dev/nvme0n1p1

#如果有swap分区   
mkswap /dev/nvme0n1p4
```

#### 挂载分区

```bash
#挂载根分区到/mnt
mount /dev/nvme0n1p2 /mnt

#创建EFI系统分区挂载点
mkdir /mnt/efi
#挂载EFI系统分区到/mnt/efi
mount /dev/nvme0n1p1 /mnt/efi

#创建home分区挂载点
mkdir /mnt/home
#挂载home分区到/mnt/home
mount /dev/nvme0n1p3 /mnt/home

```
```bash
#如果要swap分区，启用它
swapon /dev/nvme0n1p4
```

### 使用Btrfs文件系统（另一种选择）
```text
(这个方案选择esp分区挂载到/efi下，不创建home分区而是创建home子卷，分配swap文件)
```
如果你想使用Btrfs文件系统代替ext4，可以按照以下步骤操作：

#### 创建分区

```bash
cfdisk /dev/nvme0n1
```

|       设备       |       分区类型       | 分区大小 |    挂载点    |
| :------------: | :--------------: | :--: | :-------: |
| /dev/nvme0n1p1 |    Efi System    |  1G  | /mnt/efi |
| /dev/nvme0n1p2 | Linux filesystem | 剩余空间 |    /mnt   |

#### 格式化分区

```bash
#efi系统分区格式化为fat32
mkfs.fat -F 32 /dev/nvme0n1p1

#根分区使用btrfs文件系统
mkfs.btrfs /dev/nvme0n1p2
```

#### 创建Btrfs子卷

```bash
#挂载btrfs分区
mount /dev/nvme0n1p2 /mnt

#创建子卷
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@swap
#快照独立子卷
btrfs subvolume create /mnt/@snapshots
#独立的日志子卷(可选反正是，想要查日志看到底是怎么寄的就挂上)
btrfs subvolume create /mnt/@var_log

#取消挂载
umount /mnt
```

#### 挂载Btrfs子卷
> **注意**：参数有些五花八门，至于到底有没有效果确实我也不清楚，大约是有效果的吧。作用无非是以下几点。
提升性能，节省空间，延长ssd寿命，确保稳定性这样。

```bash
#常见的参数如下
```
|       参数       |       作用       |
| :------------: | :--------------: |
| subvol=(@、@home等) | 挂载目录子卷 |
| noatime | 提升读取性能，ssd延年益寿说是 |
| compress=zstd | 启用透明压缩，节省空间，延长ssd寿命，默认压缩等级为3也就是zstd:3 |

```bash
#挂载根目录子卷
mount -o noatime,compress=zstd,subvol=@ /dev/nvme0n1p2 /mnt

#挂载home子卷
mkdir /mnt/home
mount -o noatime,compress=zstd,subvol=@home /dev/nvme0n1p2 /mnt/home

#挂载snapshots子卷
#先别挂载snapshots子卷，等后面安装好快照管理工具snapper再挂载它，不然snapper这个傲娇待会还要麻烦人
# mkdir /mnt/.snapshots
# mount -o noatime,compress=zstd,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots

#挂载log子卷
mkdir -p /mnt/var/log
mount -o noatime,compress=zstd,subvol=@var_log /dev/nvme0n1p2 /mnt/var/log

#挂载EFI分区
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi


#创建并启用swap文件
#这个swap文件说是必须单独为交换文件而且要禁用写时复制COW（即交换文件必须有 +C 属性），还要关掉透明压缩参数。但实际上用btrfs filesystem创建的swap文件，默认就是禁用cow的，虽然你还能在fstab里看到有加了compress这些参数的swapfile，但是如果你用lsattr /swap/swapfile命令去查看权限，你会发现它会输出---------------C-- /swap/swapfile，所以fstab其实个人感觉不用删除哪些参数也行，不过删了也许更清晰
#至于为什么明明挂载的时候没有加compress参数但是fstab里却有参数呢，因为genfstab -U /mnt >> /mnt/etc/fstab生成挂载配置的时候它会给加上去，也许是因为内核为了方便省事，看到你其他子卷都加了，那你swap也加上算了，反正能用就行。
mkdir /mnt/swap
mount -o noatime,subvol=@swap /dev/nvme0n1p2 /mnt/swap
btrfs filesystem mkswapfile --size 4G /mnt/swap/swapfile
swapon /mnt/swap/swapfile

##如果安装后想要调整swap文件大小，只需要swapoff后，删除swap文件，创建新的合适大小的swapon文件，然后启用即可


```



## 修改镜像源

```bash
#使用vim编辑 /etc/pacman.d/mirrorlist 
vim /etc/pacman.d/mirrorlist
#在文件的最顶端添加 USTC 镜像源以加速下载
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
#刷新数据库
pacman -Sy
```

## 安装软件


```bash
#先安装内核linux-zen、固件linux firmware、基础软件包/软件包组base、base-devel、btrfs-progs
pacstrap /mnt linux-zen linux-firmware base base-devel btrfs-progs
```
> **注意**：如果文件系统是ext4，不需要装btrfs的管理工具btrfs-progs

### 安装cpu微码(根据cpu二选一)

|     类型    |      包名     |
| :-------: | :---------: |
| Intel CPU | intel-ucode |
|  Amd CPU  |  amd-ucode  |

```bash
#安装编辑器vim、网络管理器networkmanager(内置dhcp客户端和联网工具nmcli)、dhcp客户端dhcpcd(可选做备用)
#不能同时运行两个dhcp客户端
#iwd不装也行，如果你想用nmcli的话，不过装个iwd备用
#因为nmcli似乎是不支持tab补全ssid的，很蛋疼
pacstrap /mnt vim networkmanager dhcpcd amd-ucode iwd
```

## 生成fstab文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
#复查
cat /mnt/etc/fstab
#然后删了swap的compress参数(可选，不过删了更好)
vim /mnt/etc/fstab
#差不多就是这样
#/dev/nvme0n1p2 
UUID=XXXXXX /swap btrfs rw,noatime,ssd,space_cache=v2,subvol=@swap 0 0
/swap/swapfile none swap defaults 0 0

#如果没有就添加swap文件条目
echo "/swap/swapfile none swap defaults 0 0" >> /mnt/etc/fstab
#再次复查
cat /mnt/etc/fstab

```

## chroot到新系统

```bash
arch-chroot /mnt
```

## 设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 同步系统时间与硬件时间

```bash
hwclock --systohc
```

## 本地化设置

```bash
#编辑程序运行语言
vim /etc/locale.gen
#取消注释
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

#生成locale信息
locale-gen

#系统区域设置
vim /etc/locale.conf
#输入
LANG=en_US.UTF-8
```

## 网络设置

创建hostname文件

```bash
vim /etc/hostname

#编辑(自己设置一个主机名，我的主机名叫Cardcaptor)
Cardcaptor
```

本地主机名解析

```bash
#编辑hosts
vim /etc/hosts

#输入(中间空行使用tab)
127.0.0.1   localhost
::1             localhost
127.0.1.1   Cardcaptor.localdomain Cardcaptor
```

## 设置root账户密码

```bash
passwd root

#输入密码(不显示)
```

## 安装引导程序

```bash
#先安装引导加载程序grub，启动项管理器efibootmgr，以及可以让直接让快照显示在grub菜单里的工具grub-btrfs
pacman -S grub efibootmgr grub-btrfs
```

1. 安装64位的grub
2. 设置一个引导器标识(在uefi里显示该标识)，本例设置为Arch
3. 指定esp分区的挂载点为/efi
4. 将GRUB核心EFI程序 grubx64.efi 安装到 /efi/EFI/Arch，并将其模块和配置安装到 /boot/grub/x86\_64-efi/和/boot/grub/grub.cfg中

> **注意**：/boot 是微码包安装CPU微码 initramfs 文件和 mkinitcpio 安装内核与initramfs镜像的默认位置。

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=Arch
```

## 生成主配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
#之后每次安装或移除一个内核后，都要执行一次该命令来更新grub配置
```

## 完成安装并重启

```bash
#退出
exit

#先关闭swapfile，释放占用
swapoff /mnt/swap/swapfile
#如果还有占用的，直接全部取消挂载

#取消挂载
umount -R /mnt

#重启
reboot
```

