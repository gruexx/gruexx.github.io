---
title: Windows 上安装和启动 MinIO 以及设置端口
subtitle: 详细介绍如何在 Windows 上安装和配置 MinIO
date: 2024-06-21T17:00:00+08:00
draft: false
description: 本文详细介绍了在 Windows 上安装 MinIO 并进行启动和端口设置的步骤。
tags: [ "MinIO", "Windows", "安装", "配置" ]
categories: [ "教程" ]
---

# 在 Windows 上安装和启动 MinIO 以及设置端口

MinIO 是一个高性能、分布式对象存储系统，可以在 Windows 上安装和运行。以下是安装和启动 MinIO 以及设置端口的步骤：

## 1. 下载 MinIO

首先，从 MinIO 的官网或 GitHub 页面下载适用于 Windows 的 MinIO 可执行文件：

- [MinIO 官网下载链接](https://min.io/download#/windows)

## 2. 安装 MinIO

下载完成后，将可执行文件解压缩到你想要安装的位置，例如 `C:\MinIO`。

## 3. 配置 MinIO

创建一个配置文件 `minio.bat`，内容如下：

```bat
@echo off
set MINIO_ROOT_USER=<YOUR-ACCESS-KEY>
set MINIO_ROOT_PASSWORD=<YOUR-SECRET-KEY>
set MINIO_VOLUMES=D:\MinIOData
set MINIO_PORT=9000
set MINIO_CONSOLE_PORT=9001

start minio.exe server %MINIO_VOLUMES% --address :%MINIO_PORT% --console-address :%MINIO_CONSOLE_PORT%
pause
```

- `<YOUR-ACCESS-KEY>`：你的 MinIO 访问密钥。
- `<YOUR-SECRET-KEY>`：你的 MinIO 密钥。
- `D:\MinIOData`：存储数据的路径，可以根据需要更改。
- `MINIO_PORT`：设置 MinIO 服务端口，例如 `9000`。
- `MINIO_CONSOLE_PORT`：设置 MinIO 控制台端口，例如 `9001`。

## 4. 启动 MinIO

双击运行 `minio.bat` 文件，你会看到类似下面的输出：

```txt
MinIO Object Storage Server
Copyright: 2015-2021 MinIO, Inc.
License: GNU AGPLv3 <https://www.gnu.org/licenses/agpl-3.0.html>
Version: RELEASE.2023-01-01T00-00-00Z

Endpoint:  http://127.0.0.1:9000  http://192.168.1.100:9000
RootUser: <YOUR-ACCESS-KEY>
RootPass: <YOUR-SECRET-KEY>

Console: http://127.0.0.1:9001 http://192.168.1.100:9001
```

## 5. 访问 MinIO 控制台

打开浏览器，访问 MinIO 控制台：

- 控制台地址：http://127.0.0.1:9001

输入 `RootUser` 和 `RootPass` 登录。

## 6. 其他配置（可选）

如果你需要进一步配置 MinIO，可以编辑 `minio.bat` 文件中的环境变量或者通过命令行参数进行配置。更多的配置选项可以参考 MinIO
官方文档：[MinIO 官方文档](https://docs.min.io/docs/)

## 示例配置

以下是一个完整的 `minio.bat` 示例配置文件：

```bat
@echo off
set MINIO_ROOT_USER=minioadmin
set MINIO_ROOT_PASSWORD=minioadmin
set MINIO_VOLUMES=D:\MinIOData
set MINIO_PORT=9000
set MINIO_CONSOLE_PORT=9001

start minio.exe server %MINIO_VOLUMES% --address :%MINIO_PORT% --console-address :%MINIO_CONSOLE_PORT%
pause
```

## 备注

确保你的防火墙允许 MinIO 使用的端口（如 `9000` 和 `9001`）。如果需要修改这些端口，记得同时在防火墙中更新配置。
