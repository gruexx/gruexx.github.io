---
title: "树莓派5 踩坑记录 #3"
subtitle: 👀
date: 2024-01-16T17:23:20+08:00
draft: false
description: 树莓派5踩坑记录
tags: [ "树莓派" ]
categories: [ "学习笔记" ]
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

