+++
date = '2024-02-21T11:16:00+08:00'
draft = false
title = '仿真软件发光二极管闪烁实验'
slug = 'tinker-cad-linkboy-led-blink'

[taxonomies]
categories = ['note']
tags = ['dev', 'electronics']
+++

在ThinkCad/Linkboy仿真软件中完成发光二极管闪烁(Blink)

<!--more-->

## 准备工作

-   [tinkercad](http://tinkercad.com)
    
-   [linkboy](http://nuc-x.com/static/images/software/linkboy4.63-2021.8.18-win10%E5%92%8C%E6%96%B0%E7%89%88win7.rar)
    

## TinkerCad

-   tinkercad注册登陆后，依次进入 电路>创建新电路>组件>基本。拖出组件 “电阻” 与 “二极管”，并设置电阻200欧姆。找到Arduino Uno，连接电路。二极管阴极接地(GND)，电阻一端连接二极管阳极，一端连接数字端D13口，串联完成电路连接。点击右上开始模拟，二极管开始闪烁。
    

![](/images/image-4-1024x576.png)

![](/images/image-5-1024x576.png)

![](/images/image-6-1024x576.png)

![](/images/image-7-1024x576.png)

## Linkboy

-   下载并完成linkboy安装后，从开始页面进入开源硬件实验室。找到 模块>Arduino主控板系列>Arduino 主板类>型号Uno 的控制器，并拖入操作栏。接着在 电子元件系列>LED和声音类 找到 “限流电阻” 与 “红色二极管”，依次拖入。连接方式不变，参考tinkcad。连接成功后，单击arduino主控板，点击初始化，添加程序流程。拖出 指令>模块类(功能指令) 到程序框之下。点击 模块类(功能指令) 在红灯区选择“红灯点亮”与“红灯熄灭“(在流程条前的箭头快速复制命令)。再次切换到 模块>软件模块系列>定时延时类，拖出延时器。再次与”红灯点亮“与”红灯熄灭“下方分别复制两条命令，做延时条件。点击指令中的小数量，输入1s。再次进入 "指令栏"，拖出 “反复执行”循环命令，放置与初始化控制器之下
    

![](/images/image-8-1024x576.png)

![](/images/image-9-1024x576.png)

![](/images/image-10-1024x576.png)

![](/images/image-11-1024x575.png)

![](/images/image-12-1024x576.png)

![](/images/image-13-1024x576.png)

![](/images/image-14-1024x576.png)