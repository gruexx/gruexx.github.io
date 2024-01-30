---
title: My2sql解析Mysql binlog
subtitle: mysql数据库误删数据恢复记录
date: 2024-01-19T17:15:21+08:00
draft: false
tags: [ "my2sql", "go", "mysql" ]
categories: [ "笔记" ]
---

# 0 序

## Go环境安装

[Go 入门教程](https://learn.microsoft.com/zh-cn/training/modules/go-get-started/2-install-go?pivots=windows)

下载 [Go](https://go.dev/dl/) windows msi安装包并安装

查看安装的 Go 版本的详细信息

```shell
go version
```

根据教程在 `D:\Projects` 配置Go工作区

## binlog获取

mysql配置文件（通常在`/etc/mysql`）中的`[mysqld]`下的`log_bin`行查看

或者sql查询
```shell
# 是否启用了binlog
SHOW VARIABLES LIKE 'log_bin';

# 文件的路径和基本文件名
SHOW VARIABLES LIKE 'log_bin_basename';

# binlog列表
SHOW BINARY LOGS;
```
由于数据库服务器在内网，通过跳板机进行链接

```shell
# ssh连接
ssh root@xxx

# scp传输文件
scp /mnt/jxxt/mysql/data/mysql-bin.000049 root@xxx:/home
```

# 1 前言

[My2sql](https://github.com/liuhr/my2sql) 是go版MySQL binlog解析工具，通过解析MySQL binlog ，可以生成原始SQL、回滚SQL、去除主键的INSERT
SQL等，也可以生成DML统计信息。类似工具有binlog2sql、MyFlash、my2fback等，本工具基于my2fback、binlog_rollback工具二次开发而来。

# 2 安装

编译安装My2sql

```shell
cd D:/Projects/src
git clone https://github.com/liuhr/my2sql.git
cd my2sql/
go build .
```

在目录下看到生成了`my2sql.exe`文件

# 3 使用

必须的参数：

- `-user` Mysql数据库用户名
- `-password` Mysql数据库密码
- `-host` Mysql数据库地址
- `-port` Mysql数据库端口
- `-start-datetime` binlog开始时间
- `-stop-datetime` binlog结束时间
- `-start-file` 开始的binlog文件


一些能用的参数且常用的参数，详细的看[My2sql文档](https://github.com/liuhr/my2sql)：

- `-mode` repl: 伪装成从库解析binlog文件，file: 离线解析binlog文件, 默认repl
- `-local-binlog-file` 当指定-mode=file 参数时，需要指定-local-binlog-file binlog文件相对路径或绝对路径,可以连续解析多个binlog文件，只需要指定起始文件名，程序会自动持续解析下个文件
- `-databases` `-tables` 库及表条件过滤, 以逗号分隔
- `-full-columns` 生成的sql是否带全列信息，默认false
- `-ignorePrimaryKeyForInsert` 生成的insert语句是否去掉主键，默认false
- `-output-dir` 将生成的结果存放到制定目录
- `-work-type` 2sql：生成原始sql，rollback：生成回滚sql，stats：只统计DML、事务信息
- `-file-per-table` 为每个表生成一个sql文件

例子，在`D:/Projects/src/my2sql`目录下进入控制台执行：

```shell
my2sql -mode file -local-binlog-file mysql-bin.000049 -databases khjc -tables edu_record -start-datetime="2024-01-12 00:00:00" -stop-datetime="2024-01-18 16:00:00" -work-type 2sql -start-file mysql-bin.000049  -user xxx -password xxx -port xxx -host xxx -full-columns true
```

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/my2sql/output.png" caption="控制台输出" >}}

在同目录下查看输出的sql文件
