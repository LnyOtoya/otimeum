+++
date = '2024-07-18T22:38:00+08:00'
draft = false
title = 'Mpv配合UOSC实现中文菜单显示'
slug = 'mpv-uosc-chinese-menu'

[taxonomies]
categories = ['wiki']
tags = ['tools', 'mpv', 'uosc']
+++

通过安装配置uosc，使mpv拥有中文菜单显示

<!--more-->



## 首先安装mpv

```bash
sudo pacman -S mpv
```

## 接着复制mpv的配置文件到支持的几个位置，此处选择home目录下的 .config 文件夹下

```bash
cp -r /usr/share/doc/mpv/ ~/.config/
```

## 下载uosc插件

### 作者项目主页：[Github](https://github.com/tomasklaen/uosc)

### 作者项目发布页下载 uosc.conf 与 uosc.zip ,且解压到 ~/.config/mpv/ 下

![屏幕截图_20240718_223728.png](/images/屏幕截图_20240718_223728.png)

## 修改配置

```bash
vim ~/.config/mpv/scripts/uosc/main.lua
```

按下正斜杠 "/" 键入 languages 检索到第一个关键词，languages = 'slang,en', 修改为 languages = 'slang,zh-hans'， 按下esc, 键入:wq保存

![屏幕截图_20240718_224503.png](/images/屏幕截图_20240718_224503.png)

![屏幕截图_20240718_224634.png](/images/屏幕截图_20240718_224634.png)

## 大功告成

![屏幕截图_20240718_222713.png](/images/屏幕截图_20240718_222713.png)

![屏幕截图_20240718_224805.png](/images/屏幕截图_20240718_224805.png)