---
title: RSS订阅自动追番
subtitle: 实现追番自由
date: 2023-09-01T01:24:28+08:00
draft: false
description: 
tags: [ rss, 动漫, 二次元, qbittorrent]
categories: [ 教程 ]
featuredImage: "https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/dyj.jpg"
featuredImagePreview: "https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/dyj.jpg"
---

# 1 整体思路

使用**qBittorrent**的rss功能实现番剧更新时自动下载

> qBittorrent下载地址 https://www.fosshub.com/qBittorrent.html

# 2 RSS资源

大多RSS源站都被墙了，订阅了也下载不动😥

不过 [漫猫动漫](http://www.comicat.org/) 还能用，现在我主要是用这个站

> 一些资源站 https://darrendanielday.github.io/posts/anime-sites/

在搜索栏输入要看的番名，下面会出现各个字幕组的资源，这里搜索条件越精确越好

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/search.png" caption="搜索番剧">}}

点击上图红圈处的RSS链接按钮，搜索栏的就是RSS订阅链接，把他复制下来

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/rsslink.png" caption="RSS订阅链接">}}

# 3 RSS资源