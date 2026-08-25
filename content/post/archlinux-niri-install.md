+++
date = '2026-08-25T11:58:00+08:00'
draft = false
title = 'Archlinux中窗口管理器nir的安装配置'
slug = 'archlinux-niri-install'

[taxonomies]
categories = ['wiki']
tags = ['linux', 'archlinux', 'system', 'niri', 'wm']
+++

<!--more-->

# [**"时刻关注社区最新动态，一切以官方文档为准"**](https://www.archlinuxcn.org/)

## niri的安装
```bash
sudo pacman -S niri 
```
### 比较重要的程序
#### 门户
```bash
sudo pacman -S xdg-desktop-portal-gtk xdg-desktop-portal-gnome gnome-keyring
```
门户其实就类似一个翻译官，比如 NTQQ 发消息时，想要从某文件夹里选择某张表情包，就会告知门户系统，然后门户系统再统一调用文件选择器。
- xdg-desktop-portal-gtk 实现了门户的大部分功能(例如文件选择器等)
- xdg-desktop-portal-gnome 实现屏幕录制功能
- gnome-keyring 实现保存加密凭证的功能
#### 身份验证代理
实际上就是某些gui程序部分操作要root权限，就需要弹出一个输入框输入密码。
```bash
sudo pacman -S polkit-kde-agent
```
然后加入niri的配置文件，让他跟随启动
```bash
vim ~/.config/niri/config.kdl

```
```bash
spawn-at-startup "/usr/lib/polkit-kde-authentication-agent-1"
```
可以去 /usr/lib/ 目录实际验证是否存在 polkit-kde-authentication-agent-1

#### Xwayland
在niri上运行x11程序除了安装基础的xorg-wayland，还需要安装xwayland-satellite包
```bash
sudo pacman -S xorg-wayland wayland-satellite
```
也去niri配置文件里修改，加入
```bash
spawn-at-startup "xwayland-satellite"
```


### 可选程序
```bash
sudo pacman -S mako waybar swaybg swayidle swaylock fuzzel alacritty
```
- fuzzel程序启动器
- alacritty终端模拟器
#### 跟随系统启动的程序
关于niri上一些跟随系统启动的程序，有以下几种分类
##### 有标准的systemd服务
直接创建服务存放目录
```bash
mkdir -p ~/.config/systemd/user
```

例如下列程序：都有systemd服务，所以首选将其配置为systemd服务运行
- mako 通知守护程序(例如通知弹窗等)
- waybar 屏幕上的一个可自定义的状态条
- swaybg sway的桌面背景程序
- swayidle sway的空闲管理守护进程(例如空闲3分钟自动锁定屏幕等)

其中，mako和waybar开箱提供systemd服务，所以直接添加进niri会话即可
```bash
systemctl --user add-wants niri.service mako.service
systemctl --user add-wants niri.service waybar.service
```
此时查看 ~/.config/systemd/user/niri.service.wants/ 会发现
自动链接了 mako.service 和 waybar.service

其中，swaybg和swayidle没有提供systemd服务，所以需要自行创建

swaybg
```bash
vim ~/.config/systemd/user/swaybg.service 
```
编辑配置文件
```bash
[Unit]
PartOf=graphical-session.target
After=graphical-session.target
Requisite=graphical-session.target

[Service]
ExecStart=/usr/bin/swaybg -m fill -i "%h/Pictures/wallpaper/summer_light.jpg"
Restart=on-failure
```

swayidle
```bash
vim ~/.config/systemd/user/swayidle.service 
```
编辑配置文件
```bash
[Unit]
PartOf=graphical-session.target
After=graphical-session.target
Requisite=graphical-session.target

[Service]
ExecStart=/usr/bin/swayidle -w timeout 601 'niri msg action power-off-monitors' timeout 600 'swaylock -f' before-sleep 'swaylock -f'
Restart=on-failure
```
关于 swaylock ，即锁屏服务，只需要安装后绑定快捷键、图标、或者通过swayidle服务自动拉起。

最后将swaybg和swayidle也一并添加到niri会话中
```bash
systemctl --user add-wants niri.service swaybg.service
systemctl --user add-wants niri.service swayidle.service
```

如果需要移除某个服务，只需要在 ~/.config/systemd/user/niri.service.wants/ 直接删除
接着重新加载配置文件(守护进程)
```bash
systemctl --user daemon-reload
```





##### 无标准的systemd服务
直接定义在niri的配置文件中
```bash
vim ~/.config/niri/config.kdl
```
在程序前加入
```bash
spawn-at-startup "程序名"
```