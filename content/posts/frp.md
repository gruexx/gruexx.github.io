---
title: "Frp+Minio 用闲置笔记本搭建自己的OSS"
subtitle: 自己的OSS
date: 2023-08-30T17:27:59+08:00
draft: false
description: "Frp+Minio 用闲置笔记本搭建自己的OSS"
tags: [ "Frp", "Minio", "oss", "内网穿透" ]
categories: [ "学习", "教程" ]
featuredImage: "https://blog.porrizx.cc:7103/data/blog-img/frp/frp-main.jpg"
featuredImagePreview: "https://blog.porrizx.cc:7103/data/blog-img/frp/frp-main.jpg"
---

# 0 前言

由于我的博客是部署在Github Pages上，虽然文字的加载速度还可以，但是图片的加载就一言难尽了，非常影响观看体验

刚好公司电脑有公网IP，于是决定利用我闲置的笔记本搭个OSS来存放图片

# 1 准备工作

- 一个有公网IP的设备来搭建frp服务端 或 使用网上免费的frp服务器
- 一个能联网的设备来做OSS
- 一个域名

## 2 frp安装

下载frp的安装文件 https://github.com/fatedier/frp/releases/tag/v0.51.3

{{< admonition >}}
根据操作系统的不同下载不同格式的文件，但是服务端和客户端版本要保持一致
{{< /admonition >}}

### 2.1 服务端配置

我的服务端是Linux操作系统

将**frps**和**frps.ini**放在同一目录下，修改配置文件

```ini
[common]
# 绑定服务器的服务端口
bind_port = 7000
# http和https端口
vhost_http_port = 7002
vhost_https_port = 7003

# 图形界面控制台端口 以及用户名密码
dashboard_port = 7001
dashboard_user = zxy
dashboard_pwd = xxxx

# 客户端连接时验证
token = xxxx
```

执行`frps -c frps.ini`来启动服务端

可以配置成守护进程启动

```systemd
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
ExecStart=/home/frps/frps -c /home/frps/frps.ini

[Install]
WantedBy=multi-user.target
```

### 2.2 客户端配置

我的客户端是Windows操作系统

将**frps**和**frps.ini**放在同一目录下，修改配置文件