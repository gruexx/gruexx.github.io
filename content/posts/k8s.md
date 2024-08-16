---
title: k8s安装问题记录
subtitle:
date: 2024-08-16T13:39:37+08:00
lastmod: 2024-08-16T13:39:37+08:00
draft: false
tags: [ "k8s", "docker" ]
categories: [ "笔记" ]

---

## kubectl 手动下载镜像

### 查询Pod状态

```
kubectl get pod -A

```

### 查看有问题的pod的描述

```
kubectl describe pod snapshot-controller-0 -n kube-system

```

### 查看pod所在节点

```
kubectl get pod snapshot-controller-0 -n kube-system -o wide

```

### 手动拉取镜像

``` 
# 直接拉取
docker pull csiplugin/snapshot-controller:v4.0.0

# dockerhub.icu加速
docker pull dockerhub.icu/csiplugin/snapshot-controller:v4.0.0
docker image tag dockerhub.icu/csiplugin/snapshot-controller:v4.0.0 csiplugin/snapshot-controller:v4.0.0
docker rmi dockerhub.icu/csiplugin/snapshot-controller:v4.0.0
```
