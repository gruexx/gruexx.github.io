---
title: "树莓派（CM4）学习日志记录 【2-OpenCV】"
date: 2023-08-07T12:09:38+08:00
draft: false
description: "树莓派（CM4）学习日志记录 【2】"
tags: [ "树莓派" ]
categories: [ "学习" ]
featuredImage: "/img/pi2/main.jpg"
featuredImagePreview: "/img/pi2/main.jpg"
---

# 1 镜像烧录（EMMC版本）

## 1.1 下载镜像

为了方便查看摄像头，这次装个带桌面的版本 Raspberry Pi OS Lite with Desktop

## 1.2 进入烧录模式

- 首先需要将CM4 boot短接，然后通过Micro USB /Type C 接口转USB接口连接电脑，再连接电源
- 此时，电脑**设备管理器**的**通用串行总线设备**中会识别出一个BCMxxx Boot的设备
- 运行**rpiboot（引导加载程序）**，等待运行结束，在我的电脑上面会出现一个U盘的盘符

## 1.3 烧写镜像

- 使用官方的**树莓派镜像烧录器 Raspberry Pi Imager**进行烧录，镜像直接选择第一项，在设置里设置一下ssh，然后直接烧录

{{< image src="/img/pi2/sl.png" caption="非常好用的工具" src_l="/img/pi2/sl.png" >}}

## 1.4 连接桌面

系统安装完成后，可以选择直接HDMI线连接显示器就能看的桌面，或者通过VNC连接远程桌面

# 2 摄像头

## 2.1 连接摄像头

我使用的是一个免驱动的usb摄像头，所以直接连到树莓派的usb口上就行

在连接前后用`lsusb`命令查看是否有新设备上来

如何该摄像头是树莓派上唯一的视频设备那么他设备文件名会被分配为 **/dev/video0** 

## 2.2 查看视频

安装 **luvcview**
```shell
sudo apt-get install luvcview
```

查看视频
```shell
sudo luvcview
```
## 2.3 OpenCV

### 2.3.1 安装opencv

安装过程参考csdn大佬的教程安装，基本没有什么问题，主要是版本对应好就行

> [安装教程链接](https://blog.csdn.net/weixin_45911959/article/details/124157416)
