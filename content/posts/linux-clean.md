---
title: Linux 存储空间清理指南
subtitle: 高效查找和清理大文件及无用文件，保持系统流畅运行
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
4. [清理 journalctl 日志](#清理-journalctl-日志)

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

## 清理 journalctl 日志

`journalctl` 是 systemd 的日志管理工具，日志文件可能会占用大量空间，可以通过以下命令清理：

### 查看日志占用空间

```shell
# 查看 journal 日志占用的磁盘空间
sudo journalctl --disk-usage
```

### 清理旧日志

```shell
# 清理超过两周的日志
sudo journalctl --vacuum-time=2weeks

# 或者清理日志直到总大小低于 1GB
sudo journalctl --vacuum-size=1G
```

### 永久设置日志占用空间限制

```shell
# 编辑 journal 配置文件
sudo nano /etc/systemd/journald.conf

# 设置 SystemMaxUse 限制日志占用的最大空间
SystemMaxUse=1G

# 保存并退出编辑器后重启 systemd-journald 服务
sudo systemctl restart systemd-journald
```

通过以上方法，可以有效管理和清理 Linux 系统中的存储空间，确保系统平稳运行。

## MySQL 磁盘清理

1. **删除不必要的数据**:

    * 删除旧的数据表或记录。
    * 确保备份重要数据以防意外删除。

   ```
   DELETE FROM your_table WHERE condition;
   DROP TABLE old_table;
   
   
   ```

2. **优化表**: 优化表可以重组数据，释放未使用的空间。

   ```
   OPTIMIZE TABLE your_table;
   
   
   ```

3. **清理日志文件**: MySQL 生成的日志文件可能会占用大量空间，可以定期清理。

   ```
   RESET MASTER;  # 清除所有二进制日志
   PURGE BINARY LOGS BEFORE 'YYYY-MM-DD HH:MM:SS';  # 删除指定日期之前的二进制日志
   
   
   ```

4. **调整配置**: 在 `my.cnf` 配置文件中，调整以下参数以限制日志文件的大小：

   ```
   expire_logs_days = 7  # 日志文件保留7天
   max_binlog_size = 100M  # 限制单个二进制日志文件的大小为100MB
   
   
   ```
