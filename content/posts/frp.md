---
title: "Frp+Minio+Nginx 用闲置笔记本搭建自己的OSS"
subtitle: 自己的OSS
date: 2023-08-30T17:27:59+08:00
draft: false
description: "Frp+Minio 用闲置笔记本搭建自己的OSS"
tags: [ "Frp", "Minio", "oss", "内网穿透", "Nginx" ]
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
- 一个域名及SSL证书

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

将**frpc.exe**和**frpc.ini**放在同一目录下，修改配置文件

```ini
[common]
# 服务端公网ip
server_addr =  xxx.xxx.xxx.xxx
# 服务端bind_port
server_port = 7000

token = xxxx

[minio_https]
type = https
local_ip = 127.0.0.1
# 客户端跳转的端口
local_port = 81
# 域名
custom_domains = blog.porrizx.cc
```

控制台执行`frpc.exe -c frpc.ini`来启动服务端

{{< admonition >}}
域名需要解析到服务端公网ip
{{< /admonition >}}

## 3 minio安装

由于我是Windows系统，下载minio server Windows版本 http://minio.org.cn/download.shtml#/windows

```cmd
# 设置minio控制台登录用户名密码
set MINIO_ROOT_USER=xxx
set MINIO_ROOT_PASSWORD=xxx

# 启动minio 路径根据自己的修改 data是数据存放路径 控制台端口9001
C:\Users\11198\Desktop\minio\minio.exe server C:\Users\11198\Desktop\minio\data --console-address ":9001"
```

登录控制台 `localhost:9001`

## 4 nginx配置

nginx官网下载windows安装包 http://nginx.org/

修改**nginx.conf**，最关键的server配置如下：

将81端口分别转发到9000和9001端口

```conf
server {
   listen       81 ssl;
   server_name  blog.porrizx.cc;

   ssl_certificate      C:\Users\11198\Desktop\ssl\blog.porrizx.cc_bundle.pem;
   ssl_certificate_key  C:\Users\11198\Desktop\ssl\blog.porrizx.cc.key;

   ssl_session_cache    shared:SSL:1m;
   ssl_session_timeout  5m;

   ssl_ciphers  HIGH:!aNULL:!MD5;
   ssl_prefer_server_ciphers  on;

    # Allow special characters in headers
    ignore_invalid_headers off;
    # Allow any size file to be uploaded.
    # Set to a value such as 1000m; to restrict file size to a specific value
    client_max_body_size 0;
    # Disable buffering
    proxy_buffering off;
    proxy_request_buffering off;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 300;
        # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_next_upstream     http_500 http_502 http_503 http_504 error timeout invalid_header;
        chunked_transfer_encoding off;

        proxy_pass http://localhost:9001; # This uses the upstream directive definition to load balance
    }

    location ^~/data/ {
        rewrite ^/data/(.*)$ /$1 break;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-NginX-Proxy true;

        # This is necessary to pass the correct IP to be hashed
        real_ip_header X-Real-IP;

        proxy_connect_timeout 300;

        # To support websockets in MinIO versions released after January 2023
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        chunked_transfer_encoding off;

        proxy_pass http://localhost:9000; # This uses the upstream directive definition to load balance and assumes a static Console port of 9001
    }
}
```

进入**nginx.exe**所在目录，执行`start nginx`来启动nginx

## 5 使用oss

访问minio控制台 `https://域名:7003` 进行文件的管理

直接访问buckets中的图片 `https://域名:7003/data/bucket名称/图片路径`

通过将博客中的图片都替换成了oss链接，极大提升了图片的加载速度

## 6 其他

### 6.1 一种不需要安装nginx的方式

frp提供了http转https的插件，直接在配置文件中启用即可，不需要再用nginx转发

```ini
[common]
# 服务端公网ip
server_addr =  xxx.xxx.xxx.xxx
# 服务端bind_port
server_port = 7000

token = xxxx

[minio_https]
type = https
local_ip = 127.0.0.1
# 客户端跳转的端口
local_port = 9000
# 域名
custom_domains = blog.porrizx.cc

# 插件部分
plugin = https2http
plugin_local_addr = 127.0.0.1:9000

plugin_crt_path = C:\Users\11198\Desktop\ssl\blog.porrizx.cc_bundle.pem
plugin_key_path = C:\Users\11198\Desktop\ssl\blog.porrizx.cc.key
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp


# 如果还要开放其他端口，则需要再加一个二级域名，同样解析到服务端公网ip
[minio_console_https]
type = https
local_ip = 127.0.0.1
local_port = 9001
custom_domains = minio.porrizx.cc

plugin = https2http
plugin_local_addr = 127.0.0.1:9001

plugin_crt_path = C:\Users\11198\Desktop\ssl\minio\minio.porrizx.cc.pem
plugin_key_path = C:\Users\11198\Desktop\ssl\minio\minio.porrizx.cc.key
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp
```

### 6.2 免费的SSL证书申请方式

使用**FreeSSL**创建证书 https://freessl.org/create-certificate

总之一直下一步，最后验证方式选择 **DNS(CANME)** 方式会比较简单

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/frp/freessl.png" caption="选择验证方式" >}}

根据auth info创建DNS解析记录就可以了

### 6.3 minio图片访问权限设置

一般创建的bucket的权限都是private，没有权限通过链接直接访问图片

在bucket管理界面修改bucket的 **Access Policy** 设置，选择custom，然后自定义权限设置，下面是图片只读权限的配置参数

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "*"
                ]
            },
            "Action": [
                "s3:GetBucketLocation"
            ],
            "Resource": [
                "arn:aws:s3:::blog-img"
            ]
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "*"
                ]
            },
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::blog-img/*"
            ]
        }
    ]
}
```