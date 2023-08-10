---
title: "树莓派（CM4）学习日志记录 【2-OpenCV】"
date: 2023-08-07T12:09:38+08:00
draft: false
description: "树莓派（CM4）学习日志记录 【2】"
tags: [ "树莓派" ]
categories: [ "学习" ]
featuredImage: "/img/pi2/main.jpg"
featuredImagePreview: "/img/pi2/main.jpg"
---

# 1 镜像烧录（EMMC版本）

## 1.1 下载镜像

为了方便查看摄像头，这次装个带桌面的版本 Raspberry Pi OS Lite with Desktop

## 1.2 进入烧录模式

- 首先需要将CM4 boot短接，然后通过Micro USB /Type C 接口转USB接口连接电脑，再连接电源
- 此时，电脑**设备管理器**的**通用串行总线设备**中会识别出一个BCMxxx Boot的设备
- 运行**rpiboot（引导加载程序）**，等待运行结束，在我的电脑上面会出现一个U盘的盘符

## 1.3 烧写镜像

- 使用官方的**树莓派镜像烧录器 Raspberry Pi Imager**进行烧录，镜像直接选择第一项，在设置里设置一下ssh，然后直接烧录

{{< image src="/img/pi2/sl.png" caption="非常好用的工具" src_l="/img/pi2/sl.png" >}}

## 1.4 连接桌面

系统安装完成后，可以选择直接HDMI线连接显示器就能看的桌面，或者通过VNC连接远程桌面

# 2 摄像头

## 2.1 连接摄像头

我使用的是一个免驱动的usb摄像头，所以直接连到树莓派的usb口上就行

在连接前后用`lsusb`命令查看是否有新设备上来

如何该摄像头是树莓派上唯一的视频设备那么他设备文件名会被分配为 **/dev/video0**

## 2.2 查看视频

安装 **luvcview**

```shell
sudo apt-get install luvcview
```

查看视频

```shell
sudo luvcview
```

# 3 OpenCV

## 3.1 安装opencv

树莓派上安装opencv过程参考csdn大佬的教程安装，基本没有什么问题，主要是版本对应好就行

> [安装教程链接](https://blog.csdn.net/weixin_45911959/article/details/124157416)

windows下安装要简单的多

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python --no-cache-dir 

# 人脸识别训练还要装这个库
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-contrib-python --no-cache-dir
```

## 3.2 尝试使用

使用opencv自带的分类器**haarcascade_frontalface_default.xml（人脸识别）**，在安装目录下可以找到，也可以网上下载到

然后调用摄像头，可以看到能框出人脸，下面是在树莓派上运行的代码：

```python
import cv2

faceCascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')

cap = cv2.VideoCapture(0)
cap.set(3, 640)  # set Weight
cap.set(4, 480)  # set Height

while True:
    ret, img = cap.read()
    img = cv2.flip(img, 1)  # 如果摄像头倒置，将1改成-1
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    faces = faceCascade.detectMultiScale(
        gray,

        scaleFactor=1.2,
        minNeighbors=5,
        minSize=(20, 20)
    )

    for (x, y, w, h) in faces:
        cv2.rectangle(img, (x, y), (x + w, y + h), (255, 0, 0), 2)
        roi_gray = gray[y:y + h, x:x + w]
        roi_color = img[y:y + h, x:x + w]

    cv2.imshow('video', img)

    k = cv2.waitKey(30) & 0xff
    if k == 27:  # Esc for quit
        break

cap.release()
cv2.destroyAllWindows()
```
# 4 YOLO v8

要训练自己的模型，最终选用比较简单的yolo

## 4.1 安装

直接clone **yolo v8** 的代码，然后安装需要的python库

```shell
git clone https://github.com/ultralytics/ultralytics

pip install -r requirement.txt
```

## 4.2 创建数据集

在训练之前要先准备训练的图片数据，这里使用**labelImg**给图片打标记

下载 **labelImg**的release，解压后用**pycharm**打开，安装依赖的库

在运行前设置需要的标签类，方便标记的时候选择，直接在**predefined_classes.txt**文件中修改，一个单词一行

最后，运行**labelImg.py**打开窗口，在窗口左侧设置输出格式为**YOLO**

{{< image src="/img/pi2/labelImg.png" caption="labelImg 窗口" src_l="/img/pi2/labelImg.png" >}}

> [labelImg 下载地址](https://github.com/HumanSignal/labelImg/releases)
>
> **python最好用3.9版本**

标记完图片会输出一个txt文件，将其放入**labels**文件夹下，原始图片则放到**images**文件夹下

**train**是训练用的，**val**是验证用的，保证图片和txt文件名字对应就行

{{< image src="/img/pi2/coco.png" caption="数据集格式" src_l="/img/pi2/coco.png" >}}


## onnx

https://netron.app/

https://convertmodel.com/

https://github.com/nknytk/built-onnxruntime-for-raspberrypi-linux/tree/master
