---
title: "树莓派5 学习记录 #3"
subtitle: "👀"
date: 2024-01-16T17:23:20+08:00
draft: false
description: 树莓派5踩坑记录
tags: [ "树莓派", "python" ]
categories: [ "笔记" ]
---

# 0 文章索引

[树莓派(CM4) 学习记录 #1](/pi)

[树莓派4 学习记录 #2](/pi2)

# 1 python包安装问题

使用**Raspberry Pi Imager**进行烧录，但是官方给的Debian版本能在树莓派5上运行的只有最新`Debian version: 12 (bookworm)`
，无法使用更旧的版本，而Debian12的python版本是11，有些python包在[piwheels](https://www.piwheels.org/)就没有相应的版本，就比如pybluez

解决方法是使用`apt install python3-包名`全局安装

# 2 GPIO问题

之前树莓派4上python脚本里用的`rpi-gpio`来控制gpio，但是在树莓派5上用不了了

解决方法是使用`gpiozero`代替

官方文档也有从`rpi-gpio`迁移到`gpiozero`的例子

https://gpiozero.readthedocs.io/en/stable/migrating_from_rpigpio.html

# 3 镜像制作

在windows上使用 [Win32DiskImager](https://win32diskimager.org/) 来制作镜像

首先安装并打开Win32DiskImager，选择好映像文件（镜像名称自己定义）和设备，点击读取就开始制作，等进度条走完就完成了

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi3/win32.png" caption="制作镜像中" >}}

**缺点**：Win32DiskImager制作的镜像文件的大小通常与SD卡的总容量相同，而不是仅与SD卡上已用空间的大小相同。这是因为镜像工具会复制SD卡上的所有扇区，包括空白和未使用的部分。即使这些部分没有存储数据。最后导致镜像文件很大

**解决方法**：在创建镜像之前，可以手动减小SD卡上分区的大小，在使用Win32DiskImager时勾选上`仅读取已分配分区`

