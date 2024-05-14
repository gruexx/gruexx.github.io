---
title: Linux 空间清理
subtitle: Linux 空间清理
date: 2024-04-15T16:54:01+08:00
draft: false
tags: [ "my2sql", "go", "mysql" ]
categories: [ "笔记" ]
---

在 Linux 系统中，存储空间不足可能会导致系统运行缓慢或无法正常工作。本文将介绍一些方法来查找大文件和无用文件，并进行清理以释放存储空间。

## 目录

1. [查找大文件](#查找大文件)
2. [删除无用文件](#删除无用文件)
3. [自动化清理](#自动化清理)

## 查找大文件

使用 `du` 和 `find` 命令可以轻松查找系统中的大文件。

### 使用 `du` 命令

`du` 命令用于检查磁盘使用情况，可以通过以下命令找到大文件：

```shell
# 查找当前目录及子目录中最大的 10 个文件或目录
du -ah / | sort -rh | head -n 10
```

### 使用 `find` 命令

`find` 命令可以用于查找指定大小以上的文件：

```shell
# 查找大于 1G 的文件
find / -type f -size +1G
```

## 删除无用文件

系统中可能存在一些无用的缓存文件或日志文件，可以通过以下命令清理：

### 清理缓存文件

```shell
# 清理 apt 缓存
sudo apt-get clean

# 清理系统缓存
sudo sync; echo 3 > /proc/sys/vm/drop_caches
```

### 清理日志文件

系统日志文件可能会占用大量空间，可以通过以下命令清理：

```shell
# 清空所有日志文件
sudo find /var/log -type f -name "*.log" -exec truncate -s 0 {} \;
```

## 自动化清理

可以编写脚本定期清理系统中的无用文件，例如：

### 创建清理脚本

```shell
#!/bin/bash

# 清理 apt 缓存
sudo apt-get clean

# 清理系统缓存
sudo sync; echo 3 > /proc/sys/vm/drop_caches

# 清空所有日志文件
sudo find /var/log -type f -name "*.log" -exec truncate -s 0 {} \;

# 删除大于 1G 的文件（按需调整路径和大小）
find /path/to/check -type f -size +1G -exec rm -f {} \;

echo "清理完成"
```

### 设置定时任务

将脚本添加到 cron 中，定期执行：

```shell
# 编辑 cron 配置
crontab -e

# 添加以下行，每天凌晨 2 点执行清理脚本
0 2 * * * /path/to/cleanup_script.sh
```

通过以上方法，可以有效管理和清理 Linux 系统中的存储空间，确保系统平稳运行。
