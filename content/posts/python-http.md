---
title: "Python基础的 HTTP 服务器"
subtitle: "python -m http.server"
date: 2024-01-16T19:54:12+08:00
draft: false
description: 树莓派5踩坑记录
tags: [ "python" ]
categories: [ "笔记" ]
---

# 1 序

`python -m http.server` 是一个使用 Python 内置的 HTTP 服务器模块的命令。这个命令会启动一个非常基础的 HTTP
服务器，它可以用来快速地在本地网络上分享文件或者进行简单的网页测试。这个服务器是 Python 标准库的一部分，因此不需要安装任何额外的软件。

## 1.1 基本使用

- 命令格式： python -m http.server [port]
- 默认端口： 如果不指定端口，服务器默认在 8000 端口启动。
- 访问方式： 在浏览器地址栏输入 http://localhost:[port] 或 http://[IP地址]:[port] 来访问服务器。
- 文件共享： 服务器会共享命令运行的当前目录及其所有子目录的文件。

## 2.1 高级选项

- 指定端口： 可以通过添加端口号来指定服务器监听的端口，例如 python -m http.server 8080 会在 8080 端口启动服务器。
- 绑定地址： 默认情况下，服务器绑定到 0.0.0.0，这意味着它接受来自任何 IP 地址的连接。出于安全考虑，可能需要绑定到特定的地址。这可以通过
  -b 或 --bind 参数来实现，如 python -m http.server 8000 --bind 127.0.0.1。
- HTTPS 支持： Python 的 HTTP 服务器模块还支持 HTTPS。要使用 HTTPS，需要指定证书和密钥文件，可以通过 --directory 和 --cgi
  参数来启用 CGI 脚本处理。

# 2 快速搭建

创建启动脚本 `start_http_server.sh`，共享pi目录下文件

```shell
#!/bin/bash
cd /home/pi
python3 -m http.server 4000
```

确保这个脚本可执行：
```shell
chmod +x start_http_server.sh
```

创建`systemd`服务文件

```shell
sudo nano /etc/systemd/system/http_server.service
```

然后添加以下内容到该文件:

```ini
[Unit]
Description=Python HTTP Server
After=network.target

[Service]
Type=simple
User=pi
ExecStart=/home/pi/script/v8/exp/start_http_server.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

最后启动

```shell
systemctl enable http_server
systemctl start http_server
```
