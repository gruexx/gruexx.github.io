---
title: Nexus Repository部署使用记录(Linux)
subtitle: 搭建私人Maven仓库
date: 2023-11-07T10:13:35+08:00
draft: false
description: Nexus部署记录
tags: [ "Nexus", "Maven", "Linux" ]
categories: [ "笔记" ]
---

# 1 部署

## 1.1 下载安装包

下载地址 https://help.sonatype.com/repomanager3/product-information/download

找到Linux版本的安装包并下载

```shell
wget https://download.sonatype.com/nexus/3/nexus-3.61.0-02-unix.tar.gz

tar -zxvf nexus-3.61.0-02-unix.tar.gz -C /home/nexus
```

解压完成后目录下会有两个文件夹 `nexus-3.61.0-02 (nexus应用)` 和 `sonatype-work (存放数据)`

## 1.2 配置修改

在`/home/nexus/nexus-3.61.0-02/etc/nexus-default.properties`可以查看和修改nexus端口

```properties
application-port=1029
```

修改成可用的端口

## 1.3 运行

| 命令           | 说明       |
|--------------|----------|
| run          | 前台运行     |
| start        | 后台运行     |
| status       | 状态       |
| stop         | 停止       |
| restart      | 重启       |
| force-reload | 重新加载配置文件 |

执行 `/home/nexus/nexus-3.61.0-02/bin/nexus start` 后台运行Nexus

执行 `/home/nexus/nexus-3.61.0-02/bin/nexus status` 查看启动是否成功

看到输出 `nexus is running.` 代表启动成功了

访问 http://xxx.xxx.xxx.xxx:1029/ 进入Nexus主界面

## 1.4 登录

默认账号 `admin`  默认密码在 `/home/nexus/sonatype-work/nexus3/admin.password` 文件中

首次登录根据提示设置自己的密码

# 2 使用

## 2.1 第三方仓库设置

在设置中点击创建仓库

{{< figure src="https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/nexus/create_rpo.jpg" caption="创建仓库" >}}

选择 `maven(proxy)`，填写 `Name` 和 `Remote storage`

阿里云的仓库地址是 https://maven.aliyun.com/nexus/content/groups/public/

然后点击 `Create repository` 就创建好了

最后将创建的仓库加入到仓库 `maven-public` 中，这样当 `maven-public` 中找不到我们需要的jar时就会去阿里云的仓库里去找

{{< figure src="https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/nexus/public.jpg" caption="加入到仓库Members" >}}

## 2.2 上传本地jar

点击左侧 `Upload` 菜单按钮，选择仓库上传本地jar

## 2.3 项目中使用

在 `pom.xml` 引用仓库 `maven-public` 的地址

{{< figure src="https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/nexus/pom.jpg" caption="pom.xml" >}}

# 3 出现的问题

## 3.1 maven-default-http-blocker

Maven3.8.1版本使用http的仓库镜像地址会报错

`maven-default-http-blocker (http://0.0.0.0/): Blocked mirror for repositories`

最后是降了Maven版本成功解决，Maven版本由3.8.1降到了3.6.3
