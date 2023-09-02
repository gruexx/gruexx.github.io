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

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/search.png" caption="搜索番剧" >}}

点击上图红圈处的RSS链接按钮，搜索栏的就是RSS订阅链接，把他复制下来

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/rsslink.png" caption="RSS订阅链接">}}

# 3 qBittorrent使用

切换进入RSS模块，点击**新RSS订阅**按钮，将刚才复制的RSS订阅链接粘贴进去，点击确定，就能看到右边出现很多资源链接，但是我们不需要那么多

点击最右侧的 **RSS下载器...** 按钮，打开下载器设置页面

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/qbit.png" caption="新RSS订阅">}}

点击 **+** 按钮新增规则，修改**规则定义**，更进一步的筛选资源，可以根据字幕组、字体、分辨率等等信息进行过滤

再修改资源的保存目录和对哪个订阅源生效规则就可以了，后续订阅源更新后就会自动下载新的剧集

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/qbit-down.png" caption="RSS下载器规则设置">}}

# 4 后话

## 4.1 qBittorrent RSS 设置

在设置页面可以修改订阅源更新间隔和最大链接展示设置

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/rssAnimate/qbit-rss-set.png" caption="RSS设置">}}