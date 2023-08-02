---
title: "树莓派（CM4/CM4IO）学习日志记录"
date: 2023-08-02T11:50:05+08:00
draft: false
description: "树莓派（CM4）学习日志记录"
tags: [ "树莓派" ]
categories: [ "学习" ]
---

# 1 镜像烧录（EMMC版本）

## 1.1 下载镜像

这里我选择不带桌面的版本 Raspberry Pi OS Lite

[镜像下载](https://www.raspberrypi.com/software/operating-systems/)

## 1.2 进入烧录模式

- 首先需要将CM4 boot短接，然后通过Micro USB /Type C 接口转USB接口连接电脑，再连接电源
- 此时，电脑**设备管理器**的**通用串行总线设备**中会识别出一个BCMxxx Boot的设备
- 运行**rpiboot（引导加载程序）**，等待运行结束，在我的电脑上面会出现一个U盘的盘符
- 使用**SDFormatter**格式化EMMC
- 烧录的准备工作就完成了！！！

## 1.3 烧写镜像

- （不推荐）使用**Win32DiskImager**烧写镜像，需要将下载的 .xz 结尾的镜像文件解压得到 .img 结尾的镜像文件，然后进行烧写
- （推荐）使用官方的**树莓派镜像烧录器 Raspberry Pi Imager**进行烧录，可以直接使用 .xz
  文件烧录，还能设置主机名、开启ssh服务、ssh登录用户名密码、wifi、语音
- 烧录完毕之后，在我的电脑会识别出一个U盘的盘符
- 如果是使用**Win32DiskImager**烧写镜像，还需要开启下ssh服务和设置登录的用户名密码：
- 在根目录下新建一个名称为**ssh**的文件（没有后缀名）来开启ssh服务
- 创建 userconf.txt文件，该文件内输入用户名和密码的密文，以下例子，用户名为pi，密码为raspberry的密文

```
pi:$6$4ilokQRQxmURT.py$aJWBQ5yniJJPwV3CKawYJcnSK5JZGhrVZYF3K4omRUFv6KL0MysEH7F4NZRMNMcYF.U3xsQvWrx7ZL2GKxuv.1
```

> 下载 [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

## 1.4 烧录完成并重启

- 烧录完毕断开电源，断开和电脑的连接线，将BOOT断开，重新上电
- 连接网线，然后使用shell工具连接树莓派

# 2 系统软件安装

- `sudo passwd root` 设置root密码

## 2.1 安装wiringpi

```shell
# 选择要保存的目录
cd download 
 
# 下载deb包
wget https://github.com/WiringPi/WiringPi/releases/download/2.61-1/wiringpi-2.61-1-armhf.deb
 
# 安装deb包
sudo dpkg -i wiringpi-latest.deb

# 查看版本
gpio -v

# 查看gpio信息
gpio readall

# 查看板子详细信息
pinout
```
