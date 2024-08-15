---
title: "Vsftpd 安装使用"
date: 2023-08-01T09:42:12+08:00
draft: false
description: "如何使用vsftpd创建ftp服务器"
tags: [ "vsftpd", "教程" ]
categories: [ "笔记" ]
featuredImage: "https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/vsftpd/ftp.jpg"
featuredImagePreview: "https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/vsftpd/ftp.jpg"
---

# 1 关于 vsftpd

> vsftpd（Very Secure FTP Daemon）是一个高度安全的FTP服务器软件，
> 被广泛用于Linux和类Unix系统。它旨在提供一个安全可靠的FTP服务器，
> 具有良好的性能和一系列功能。

## 1.1 安装vsftpd软件包 （环境ubuntu）

```shell
sudo apt update
sudo apt install vsftpd
```

## 1.2 配置vsftpd

1.2.1 配置文件位于 /etc/vsftpd.conf。你可以使用文本编辑器（如Nano或Vi）来编辑该文件

```shell
sudo nano /etc/vsftpd.conf
```

1.2.2 确保以下设置在配置文件中启用

```
write_enable=YES
local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES
```

## 1.3 重启vsftpd服务

```shell
sudo service vsftpd restart
```

## 1.4 配置防火墙

如果你的Ubuntu服务器启用了防火墙（如ufw），请确保开放FTP端口 21：

```shell
sudo ufw allow 21/tcp
sudo ufw enable
# 查看防火墙状态
sudo ufw status
```

# 2 创建专门的FTP用户并限制其主目录的访问权限

## 2.1 创建FTP用户

使用adduser命令创建新的FTP用户。在这里，我们假设你要创建一个名为"ftpuser"的FTP用户。执行以下命令并按照提示设置密码和其他信息：

```shell
sudo adduser ftpuser
```

## 2.2 限制FTP用户主目录的访问权限

默认情况下，新创建的用户的主目录位于/home/ftpuser，我们可以更改其主目录并限制其访问权限。首先，创建一个新的目录用于FTP用户的主目录。例如，我们在/ftp目录下创建一个名为"
ftpuser_home"的目录，并将其所有权赋予"ftpuser"用户：

```shell
sudo mkdir /ftp/ftpuser_home
sudo chown ftpuser:ftpuser /ftp/ftpuser_home
```

## 2.3 修改FTP用户的主目录

编辑/etc/passwd文件，将FTP用户的主目录修改为上一步创建的目录/ftp/ftpuser_home。可以使用usermod命令来完成此操作：

```shell
sudo usermod -d /ftp/ftpuser_home ftpuser
```

## 2.4 限制FTP用户登录

要限制FTP用户仅能使用FTP服务而无法登录系统，可以在/etc/passwd文件中将其登录Shell设置为/usr/sbin/nologin。可以使用usermod命令完成此操作：

```shell
sudo usermod -s /usr/sbin/nologin ftpuser
```

## 2.5 重启vsftpd服务

完成配置后，重启vsftpd服务使更改生效：

```shell
sudo service vsftpd restart
```

# 3 出现的问题记录

## 3.1 vsftpd登录报530

问题：vsftpd登录报530 Login incorrect无法登录问题

解决方法：进入/etc/pam.d将vsftpd文件中的pam_shells.so改为pam_nologin.so，然后systemctl restart vsftpd重启服务，连接ftp成功

>[问题参考的文章链接](https://www.cnblogs.com/lipanchn/p/11783518.html#:~:text=%E8%A7%A3%E5%86%B3%EF%BC%9A%20%E8%BF%9B%E5%85%A5%2Fetc%2Fpam.d%E5%B0%86vsftpd%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84pam_shells.so%E6%94%B9%E4%B8%BApam_nologin.so%EF%BC%8C%E7%84%B6%E5%90%8Esystemctl,restart%20vsftpd%E9%87%8D%E5%90%AF%E6%9C%8D%E5%8A%A1%EF%BC%8C%E8%BF%9E%E6%8E%A5ftp%E6%88%90%E5%8A%9F)
