---
title: Mqtt 安装使用记录
subtitle: 笔记自用
date: 2023-09-06T10:30:59+08:00
draft: false
description: Mqtt 安装使用
tags: [ "Mqtt" ]
categories: [ "笔记" ]
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

图形界面操作 比较简单 略

## 2.2 用python向mqtt发数据

下面是一个例子，从串口接受数据并发送到mqtt主题

```python
import serial
import paho.mqtt.client as mqtt

ser = serial.Serial('/dev/ttyAMA2', 115200)  # Serial port name and baud rate
mqtt_client = mqtt.Client()
mqtt_client.username_pw_set('xxx', 'xxx')  # MQTT username and password


def on_connect(client, userdata, flags, rc):
    print('Connected to MQTT broker with result code ' + str(rc))


mqtt_client.on_connect = on_connect
mqtt_client.connect('xxx.xxx.xxx.xxx', 1883)  # MQTT server IP address and port number
mqtt_client.loop_start()
while True:
    data = ser.read(1024)  # Read serial data and decode it
    # print(data)  # Print data to console
    mqtt_client.publish('pi-test', data)  # Publish data to MQTT topic
```

{{< admonition tip>}}
python连接MQTT连接错误码：

- rc=0：连接成功
- rc=1：协议版本不接受
- rc=2：无效客户端标识符被拒绝
- rc=3：服务不可用
- rc=4：用户名或密码错误
- rc=5：无权连接
{{< /admonition >}}
