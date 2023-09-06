---
title: Mqtt 安装使用
subtitle: 服务端客户端安装记录
date: 2023-09-06T10:30:59+08:00
draft: false
description: Mqtt 安装使用
tags: [ "Mqtt" ]
categories: [ "学习笔记" ]
---

# 1 服务端

## 1.1 Linux系统

Ubuntu下直接 `app-get install mosquitto` 安装

配置用户名密码 `mosquitto_passwd -c /etc/mosquitto/pwfile.example porrizx`

修改配置文件 `/etc/mosquitto/mosquitto.conf`

新增下面几行

```conf
# mqtt端口
listener 1883
# 用户名密码配置文件
password_file /etc/mosquitto/pwfile.example
```

## 1.2 Windows系统

下载并安装mosquitto `https://mosquitto.org/download/`

进入`mosquitto.exe`所在目录，修改配置文件（同上）

修改完成后在控制台执行 `mosquitto.exe -c mosquitto.conf -v` 启动

# 2 客户端

## 2.1 MQTTX

下载安装MQTTX https://mqttx.app/zh

## 2.2 
