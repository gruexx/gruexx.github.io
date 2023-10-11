---
title: Sqlite3 安装使用记录
subtitle: 笔记自用
date: 2023-10-09T09:46:57+08:00
draft: false
description: "Sqlite3"
tags: [ "Sqlite3", "Linux", "Python" ]
categories: [ "学习笔记" ]
---

# 安装

```shell
sudo apt-get install sqlite3
```

安装失败 `sqlite3 : Depends: libsqlite3-0 (= 3.27.2-3+deb10u2) but 3.34.1-3 is to be installed`

解决方式

```shell
sudo dpkg --purge --force-depends libsqlite3-0
sudo apt-get install libsqlite3-0
sudo apt-get install -f
sudo apt-get install libsqlite3-dev
```

# 数据库操作

```python
# 连接到SQLite3数据库
conn = sqlite3.connect('data.db', timeout=10)
# 创建一个游标对象来执行SQL
cursor = conn.cursor()
# 执行SQL
cursor.execute("update kettle set duration = ? where id = 1", (duration,))
# 提交
conn.commit()
# 关闭游标
cursor.close()
# 关闭连接
conn.close()
```
