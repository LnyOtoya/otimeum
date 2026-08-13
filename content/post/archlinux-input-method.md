+++
date = '2024-10-02T10:44:00+08:00'
draft = false
title = 'Archlinux配置输入法方案'
slug = 'archlinux-input-method'

[taxonomies]
categories = ['wiki']
tags = ['linux', 'archlinux', 'tools', 'fcitx5', 'rime']
+++

fcitx5和Rime的安装配置

<!--more-->



## 框架安装

直接安装fcitx5-im包组 ，它包含fcitx5本体以及其他配置模块

```bash
sudo pacman -S fcitx5-im
```

## 引擎安装

可以选择fcitx5-chinese-addons包 或者fcitx5-rime包 本篇采用Rime引擎，其自定义程度较高。

```bash
sudo pacman -S fcitx5-rime
```

## 配置

本篇使用Kde plasma + Wayland,其他环境可以参考[Archwiki](https://wiki.archlinuxcn.org/wiki/Fcitx5)或者[FcitxWiki](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma)

kde用户可以进入 系统设置 > 键盘 >虚拟键盘, 勾选Fcitx 5 Wayland 启动器

接着，为Xwayland程序设置变量

```bash
sudo vim /etc/environment
```

配置

```bash
XMODIFIERS=@im=fcitx
```

保存后注销或者重启，默认ctrl + space切换输入法

对于输入法方案，可以使用雾凇拼音，即rime-ice， 虽然个人使用体验来看都差不多(bushi

可以使用yay安装

```bash
yay rime-ice-git
```

或者克隆其仓库

```bash
git clone https://github.com/iDvel/rime-ice.git
```

定义Rime配置文件夹位置于 ~/.local/share/fcitx5/rime/

此外，对于 chromium/electron 应用程序，需要在启动命令后加上参数 --enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime 来解决输入法漏字，字母上屏的问题

拿qq来举例，安装linuxqq后，编辑其desktop文件

```bash
sudo vim /usr/share/applications/qq.desktop
```

接着编辑Exec行，加入参数

```bash
Exec=linuxqq %U --enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime
```

保存后重新启动应用，便不会再出现漏字的情况。