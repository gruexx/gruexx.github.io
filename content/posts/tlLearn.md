---
title: "延时摄影视频制作教程"
date: 2023-08-20T21:01:00+08:00
draft: false
description: "延时摄影视频制作教程"
tags: [ "延时摄影", "视频", "教程", "LRTimelapse", "Lightroom", "DaVinci" ]
categories: [ "摄影","学习" ]
featuredImage: "https://blog.porrizx.cc:7103/blog-img/tlLearn/TL-383.jpg"
featuredImagePreview: "https://blog.porrizx.cc:7103/blog-img/tlLearn/TL-383.jpg"
---

# 0 前置准备

照片处理：LRTimelapse、Lightroom

视频制作：DaVinci或者Pr

# 1 照片处理

打开**LRTimelapse**，选择照片所在文件夹

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/find_dir.png" caption="照片所在文件夹" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/find_dir.png" >}}

设置关键帧，根据照片数量以及照片内容设置关键帧数量，如果照片明暗变化比较大，可以多设点

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/keyframe_set.png" caption="设置关键帧" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/keyframe_set.png" >}}

设置完成后点击保存，然后点击**Drag to Lightroom**拖到Lightroom里面

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/into_lrt.png" caption="Drag to Lightroom" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/into_lrt.png" >}}

点击**导入**，点击右下角选择 **01 LRT Keyframes**，选完这里照片数量就是之前设置的关键帧的数量

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/lrt_keyframe.png" caption="Drag to Lightroom" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/lrt_keyframe.png" >}}

接下来就是**修改照片**，调整照片参数到自己满意的程度，然后全选图片，可以点击 **01 LRTimelapse Sync Keyframes**
同步所有的关键帧样式，也可以继续调整每个关键帧

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/sync.png" caption="同步所有关键帧设置" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/sync.png" >}}

调整完，将元数据存储到文件

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/save_main_data.png" caption="将元数据存储到文件" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/save_main_data.png" >}}

回到**LRTimelapse**，点击**Reload**

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/reload.png" caption="reload" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/reload.png" >}}

接下来的步骤就是：**Auto Transition** -> **Visual Previews** -> **Visual Deflicker**

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/at_vp_vd.png" caption="自动过渡->视觉预览->视觉去闪烁" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/at_vp_vd.png" >}}

视觉去闪烁设置，对于曲线比较平滑的可以设少点

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/deflicker.png" caption="视觉去闪烁设置" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/deflicker.png" >}}

完成后**保存**

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/save.png" caption="lrt 保存" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/save.png" >}}

再回到**Lightroom**，从文件中读取元数据

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/read_main_data.png" caption="从文件中读取元数据" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/read_main_data.png" >}}

**导出**照片

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/export.png" caption="导出" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/export.png" >}}

照片处理部分就完成了

# 2 视频制作

打开**DaVinci**，新建一个项目，修改下项目设置，主要是分辨率和帧率

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/project_set.png" caption="项目设置" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/project_set.png" >}}

将导出的图片全选然后直接拖到时间线上，可以调整下缩放去除两侧黑边，剩下就自由发挥

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/video_make.png" caption="直接拖到时间线" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/video_make.png" >}}

完成后可以渲染了

{{< image src="https://blog.porrizx.cc:7103/blog-img/tlLearn/delivery.png" caption="交付" src_l="https://blog.porrizx.cc:7103/blog-img/tlLearn/delivery.png" >}}

成片效果如下

{{< bilibili id=BV1Lp4y1K7gp >}}
