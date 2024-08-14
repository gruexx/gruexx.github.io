---
title: Linux 存储空间清理指南
subtitle: 高效查找和清理大文件及无用文件，保持系统流畅运行
date: 2024-04-15T16:54:01+08:00
lastmod: 2024-08-14T15:41:01+08:00
draft: false
tags: [ "my2sql", "go", "mysql" ]
categories: [ "笔记" ]

---

在 Linux 系统中，存储空间不足可能会导致系统运行缓慢或无法正常工作。本文将介绍一些方法来查找大文件和无用文件，并进行清理以释放存储空间。

## 查找大文件

使用 `du` 和 `find` 命令可以轻松查找系统中的大文件。

### 使用 `du` 命令

`du` 命令用于检查磁盘使用情况，可以通过以下命令找到大文件：

```shell
# 查看当前目录所以文件或目录的大小
du -sh * 

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

## ClickHouse 磁盘清理

### 查询表数据的占用情况

要查看每个表在磁盘上的占用情况，可以使用以下 SQL 查询：

```
SELECT database,
       table,
       sum(bytes_on_disk)                     as bytes,
       formatReadableSize(sum(bytes_on_disk)) AS size_on_disk,
       sum(rows)                              AS total_rows,
       count()                                AS parts_count
FROM system.parts
WHERE active
GROUP BY database,
         table
ORDER BY bytes desc;



```

这个查询会显示每个数据库中的每个表的磁盘使用量、表的总行数以及活跃的分片（parts）的数量。

查询单个表的分区情况，例如：

```
SELECT * FROM system.parts where database = 'system' and `table`= 'query_log';

```

### 手动清理

对于system库log表数据，可以直接根据event_date删除：

```
ALTER TABLE system.query_log DELETE WHERE event_date < '2024-08-01';
```

或者直接删除分区，效率更高：

```
alter table system.query_log drop partition '202408';

```

### 自动清理和设置阈值

#### **TTL（Time to Live）设置**

你可以为表设置 TTL，自动删除超过一定时间的数据。这是 ClickHouse 提供的一种机制，可以定期清理数据，保持表的体积在可控范围内。

```
ALTER TABLE system.query_log MODIFY TTL event_date + INTERVAL 30 DAY;

```

这条命令将确保 `event_date` 超过 30 天的数据自动删除。

## docker log清理

Docker 容器的 JSON 日志文件可以随着容器运行时间的增加而占用大量磁盘空间。你可以通过以下方法来清理这些日志文件并设置大小限制。

### 1. 清理现有 JSON 日志文件

Docker 默认将容器的日志存储在 `/var/lib/docker/containers/<container_id>/`
目录下，每个容器的日志文件名为 `<container_id>-json.log`。

#### **手动清理日志**

你可以手动清理这些日志文件：

```
# 找到日志文件路径
docker inspect --format='{{.LogPath}}' <container_id>

# 清空日志文件
truncate -s 0 /var/lib/docker/containers/<container_id>/<container_id>-json.log


```

#### **删除日志文件**

或者直接删除日志文件（Docker 会自动创建新的日志文件）：

```
rm /var/lib/docker/containers/<container_id>/<container_id>-json.log


```

### 2. 设置日志大小限制

为了避免日志文件无限制地增长，可以通过 Docker 的日志驱动和配置选项来限制日志文件的大小和轮换（rotation）。

#### **在 Docker 守护进程配置中设置全局限制**

修改 Docker 的守护进程配置文件（通常是 `/etc/docker/daemon.json`），添加日志大小限制和轮换配置：

```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",   // 单个日志文件的最大大小
    "max-file": "3"      // 保留的日志文件的数量
  }
}


```

此配置示例将每个容器的日志文件限制为 10MB，并最多保留 3 个日志文件。这意味着一旦日志文件达到 10MB，Docker
将轮换日志文件，创建一个新的日志文件，并删除最旧的日志文件。

配置文件修改后，重新启动 Docker 服务以使更改生效：

```
sudo systemctl restart docker


```

#### **为特定容器设置日志限制**

你也可以在启动容器时指定日志限制，而不是全局设置：

```
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  your_image_name


```

这种方式可以为特定容器定制日志文件的大小和轮换策略。

### 3. 定时清理日志

即使设置了日志限制，你可能仍然希望定期清理日志或检查磁盘使用情况。可以通过设置定时任务来清理日志文件：

1. **编写清理脚本**：

```
#!/bin/bash
# 查找并清理所有 Docker 容器的日志文件
find /var/lib/docker/containers/ -name "*-json.log" -exec truncate -s 0 {} \;


```

2. **设置 `cron` 定时任务**： 将此脚本设置为定期运行，比如每天凌晨 3 点运行一次：

```
crontab -e


```

添加以下行：

```
0 3 * * * /path/to/cleanup_script.sh


```

### 4. 监控和预警

结合监控工具（如 Prometheus + Grafana）监控 Docker
容器的磁盘使用情况，并在日志文件大小过大或磁盘空间不足时发出警报。可以设置监控指标来跟踪 `/var/lib/docker/containers/`
目录的大小。

通过这些措施，你可以有效管理和控制 Docker 容器的日志文件大小，避免日志文件无限增长导致的磁盘空间耗尽问题。

## docker overlay2清理

Docker 使用 `overlay2` 作为其存储驱动程序之一，它是 Docker 的默认存储驱动程序，用于管理容器的分层文件系统。理解 `overlay2`
以及如何清理它对于优化 Docker 环境的磁盘使用非常重要。

### 1. 什么是 `overlay2`？

`overlay2` 是 Docker
的存储驱动程序，用于支持容器的分层文件系统。它将不同的层次叠加在一起，每一层都可以包含文件和目录的修改、添加或删除。`overlay2`
存储驱动的主要优点包括高效的磁盘使用和快速的容器启动时间。

在 `overlay2` 文件系统中，每个 Docker 镜像层和容器层都被存储为只读或可写层。每个容器都会创建自己的可写层，所有的写操作（如文件创建、修改）都会在这个层上进行，而不会影响到基础镜像层。

### 2. `overlay2` 的文件结构

`overlay2` 的数据通常存储在 `/var/lib/docker/overlay2/` 目录下，每个子目录对应一个镜像层或容器的层。`overlay2`
目录结构通常包括以下内容：

* **`diff/`**：存放每个镜像层的文件和目录。
* **`merged/`**：存放已经叠加的文件系统，这些是容器运行时看到的最终文件系统。
* **`work/`**：存放工作目录，用于临时存放 `overlay2` 操作中的中间状态。
* **`upper/`**：容器的可写层，存放容器运行时修改或新增的文件。

### 3. 清理 `overlay2` 数据

由于 Docker 容器的镜像和层存储在 `overlay2` 中，清理不必要的数据可以释放大量的磁盘空间。清理 `overlay2` 主要包括以下几个步骤：

#### 删除未使用的 Docker 容器

首先，删除已经停止并且不再需要的容器：

```
docker container prune


```

这个命令将删除所有停止的容器。

#### 删除未使用的 Docker 镜像

删除不再需要的 Docker 镜像可以释放大量空间：

```
docker image prune


```

这个命令将删除未使用的悬空镜像（dangling images），即那些没有被任何容器使用的镜像层。

如果你想删除所有未被使用的镜像，可以使用以下命令：

```
docker image prune -a


```

这个命令将删除所有未被任何容器使用的镜像，而不仅仅是悬空镜像。

#### 删除未使用的 Docker 网络

网络在 Docker 中也会占用一些空间，虽然通常不会很多，但你仍然可以定期清理它们：

```
docker network prune


```

#### 删除未使用的 Docker 卷

卷用于存储数据，即使相关的容器被删除，卷仍然可能保留。如果这些卷不再需要，可以删除它们：

```
docker volume prune


```

#### 手动清理 `overlay2`

一般来说，清理容器、镜像、卷和网络之后，`overlay2` 目录会自动清理掉不再使用的层。但是，如果你仍然发现 `overlay2`
占用了大量空间，可能需要手动检查并清理不必要的文件。

你可以手动检查 `/var/lib/docker/overlay2/` 目录下的文件和目录，找到那些不再使用的层进行清理。然而，这通常不推荐，因为手动删除可能导致
Docker 工作不正常。如果你怀疑 Docker 占用了不必要的空间，可以尝试重新启动 Docker 服务，这通常会帮助释放已删除对象的空间。

### 4. 设置自动清理策略

为了防止 `overlay2` 目录不断增长并耗尽磁盘空间，可以设置定期的清理任务：

* **使用 `cron` 定期运行清理命令**：

  创建一个清理脚本，比如 `docker_cleanup.sh`：

  ```
  #!/bin/bash
  docker container prune -f
  docker image prune -f -a
  docker volume prune -f
  docker network prune -f
  
  
  ```

  将其加入到 `cron` 中，使其定期运行，例如每天凌晨 2 点：

  ```
  crontab -e
  
  
  ```

  添加以下行：

  ```
  0 2 * * * /path/to/docker_cleanup.sh
  
  
  ```

通过这些步骤，你可以有效管理 Docker 的 `overlay2` 存储空间，防止其占用过多的磁盘空间。
