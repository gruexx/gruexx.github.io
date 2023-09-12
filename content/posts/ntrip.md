---
title: Ntrip问题记录
subtitle:
date: 2023-09-12T17:13:10+08:00
draft: false
description: Mqtt 安装使用
tags: [ "ntrip" ]
categories: [ "学习笔记" ]
---

# 1 如何将挂载点挂载到caster

`login_request` 写的是否正确很关键

```python
import socket

# Connect to NTRIP server
ntrip_host = 'xxx.xxx.xxx.xxx'
ntrip_port = xx
ntrip_mount = 'xx'
ntrip_password = 'xx'
ntrip_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

while True:
    try:
        ntrip_socket.connect((ntrip_host, ntrip_port))
        # Send login request to NTRIP server
        login_request = "SOURCE " + ntrip_password + " /" + ntrip_mount + "\r\nSource-Agent:NTRIP Caster 2.0.21/2.0\r\n\r\n"
        ntrip_socket.sendall(login_request.encode())

        while True:
            data = ntrip_socket.recv(1024)
            if data:
                print(data)
    except socket.error as e:
        print('Socket error: {}'.format(e))
        ntrip_socket.close()
```
# 2 如何从挂载点获取数据

`request` 写的是否正确很关键，其中授权部分是 【**用户名:密码**】的字符串转BASE64

```python
import base64
import socket
from pyrtcm import RTCMMessage


def string_to_base64(input_string):
    # 将字符串转换为字节串
    input_bytes = input_string.encode('utf-8')

    # 使用 base64 编码将字节串转换为 Base64 编码
    base64_encoded = base64.b64encode(input_bytes)

    # 将字节串转换为字符串并返回
    return base64_encoded.decode('utf-8')


def get_ntrip_data(mountpoint, ntrip_server, port, username, password):
    # 构建 NTRIP 请求信息
    url = f"/{mountpoint}\r\n"
    auth = string_to_base64(f"{username}:{password}")
    authorization = f"Authorization: Basic {auth}\r\n"

    # 创建 TCP socket
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.connect((ntrip_server, port))

            # 构建 NTRIP 请求消息
            request = f"GET {url}HTTP/1.1\r\nUser-Agent: NTRIP GNSSInternetRadio/1.4.10\r\nAccept: */*\r\nConnection: close\r\n{authorization}\r\n"
            print(request)
            s.sendall(request.encode())

            # 读取和处理响应数据
            while True:
                data = s.recv(2048)
                rtcm_message = RTCMMessage(data)
                if rtcm_message:
                    print(rtcm_message)

    except socket.error as e:
        print(f"Error: {e}")


if __name__ == "__main__":
    # NTRIP 服务器信息
    ntrip_server = "xxx.xxx.xxx.xxx"
    port = xx  # 默认 NTRIP 端口为 2101

    # 挂载点信息
    mountpoint = "xx"
    username = "xx"
    password = "xx"

    # 获取挂载点的数据
    get_ntrip_data(mountpoint, ntrip_server, port, username, password)

```
