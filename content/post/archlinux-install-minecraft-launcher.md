+++
date = '2024-03-11T11:27:00+08:00'
draft = false
title = 'Archlinux安装Mincrosoft启动器'
slug = 'archlinux-install-minecraft-launcher'

[taxonomies]
categories = ['wiki']
tags = ['linux', 'archlinux', 'tools', 'minecraft']
+++

安装配置mc启动器hmcl

<!--more-->



## 安装openjdk17

```bash
yay -S jdk17-openjdk
# jre17-openjdk-headless jre17-openjdk 
```

## 下载hmcl

在作者Github发行页 : [https://github.com/HMCL-dev/HMCL/releases](https://github.com/HMCL-dev/HMCL/releases) 中，下载最新的jar包，且放在合适的文件夹中 本例将其放于 /home/otoya/Downloads/Games/ 文件夹内

然后下载启动器图标 : [hmcl.png](https://github.com/HMCL-dev/HMCL/blob/main/HMCL/image/hmcl.png)

![hmcl.png](/images/hmcl.png)

将其放入 /usr/share/icons/hicolor/64x64/apps/ 文件夹内

## 为hmcl.jar文件创建程序快捷方式

切换进入存放应用程序项目的目录

```bash
cd /usr/share/applications/
```

创建新项目

```bash
vim hmcl.desktop
```

编辑该文件

```ini
[Desktop Entry]

# type一般为Application
Type=Application

# 本文件所遵循的桌面项规范版本(可选)
Version=1.0

# 应用程序的名称
Name=Hmcl

# 文件目录(刚才下载的jar包存放的路径,自定义)
Path=/home/otoya/Downloads/Games/

# 可执行文件，可以带参(HMCL-3.5.6.240.jar为下载的jar包名)
# 可以通过这种方法更改其他程序启动时执行参数
Exec=java -jar HMCL-3.5.6.240.jar

# 图标名称
Icon=hmcl

# 应用程序是否需要运行在终端中
Terminal=false

# 本桌面项将显示在哪些分类中
Categories=Game;
```

> **Minecraft启动器安装**：完成

![mc.png](/images/mc.png)

![mc2.png](/images/mc2.png)