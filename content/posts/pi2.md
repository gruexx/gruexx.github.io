---
title: "树莓派4 学习记录 #2"
subtitle: 基于摄像头的目标检测尝试
date: 2023-08-07T12:09:38+08:00
lastmod: 2023-09-05T11:04:38+08:00
draft: false
description: "树莓派（CM4）学习日志记录 【2-OpenCV Yolov5 摄像头 目标检测】"
tags: [ "树莓派", "Yolov5",  "OpenCV", "摄像头", "目标检测", "onnx", "onnxruntime" ]
categories: [ "笔记" ]
featuredImage: "https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/main.jpg"
featuredImagePreview: "https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/main.jpg"
---

# 0 文章索引

[树莓派(CM4) 学习记录 #1](/pi)

[树莓派5 踩坑记录 #3](/pi3)

# 1 镜像烧录（EMMC版本）

## 1.1 下载镜像

为了方便查看摄像头，这次装个带桌面的版本 Raspberry Pi OS Lite with Desktop

## 1.2 进入烧录模式

- 首先需要将CM4 boot短接，然后通过Micro USB /Type C 接口转USB接口连接电脑，再连接电源
- 此时，电脑**设备管理器**的**通用串行总线设备**中会识别出一个BCMxxx Boot的设备
- 运行**rpiboot（引导加载程序）**，等待运行结束，在我的电脑上面会出现一个U盘的盘符

## 1.3 烧写镜像

- 使用官方的**树莓派镜像烧录器 Raspberry Pi Imager**进行烧录，镜像直接选择第一项，在设置里设置一下ssh，然后直接烧录

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/sl.png" caption="非常好用的工具" >}}

{{< admonition tip "提醒" >}}
不要忘了点击右下角的齿轮设置ssh用户名和密码
{{< /admonition >}}

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

# 4 YOLO v5

要训练自己的模型，最终选用比较简单的yolo，考虑到树莓派的性能，选择v5版本

## 4.1 安装

直接clone **yolo v5** 的代码，然后安装需要的python库

```shell
git clone https://github.com/ultralytics/yolov5.git

pip install -r requirement.txt
```

## 4.2 创建数据集

在训练之前要先准备训练的图片数据，这里使用**labelImg**给图片打标记

下载 **labelImg**的release，解压后用**pycharm**打开，安装依赖的库

在运行前设置需要的标签类，方便标记的时候选择，直接在**predefined_classes.txt**文件中修改，一个单词一行

最后，运行**labelImg.py**打开窗口，在窗口左侧设置输出格式为**YOLO**

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/labelImg.png" caption="labelImg 窗口" >}}

> [labelImg 下载地址](https://github.com/HumanSignal/labelImg/releases)
>
> **python最好用3.9版本**

标记完图片会输出一个txt文件，将其放入**labels**文件夹下，原始图片则放到**images**文件夹下

**train**是训练用的，**val**是验证用的，保证图片和txt文件名字对应就行

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/coco.png" caption="数据集格式" >}}

## 4.3 训练

找到yolov5项目目录下的**train.py**，修改几个主要的参数:

1. **'--weights'**  基于该模型训练，在yolov5 github页面下载
2. **'--cfg'** yolov5n.yaml 项目自带的模板 只需要修改文件中类的数量，即nc的值
3. **'--data'** coco128.yaml 项目自带的模板 主要修改数据集的路径和类的名称
4. **'--epochs'** 训练的轮次
5. **'--imgsz'** 图片尺寸 通常是 320或640

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/train.png" caption="train.py" >}}

修改完直接运行开始训练

> [Model yolov5n.pt 下载地址](https://github.com/ultralytics/yolov5/releases/download/v7.0/yolov5n.pt)

## 4.4 训练完成

训练完成后在项目的**runs**目录下回输出训练结果

找到**weight**文件夹，这里存放的是pyTorch格式的权重文件

**best.pt**是最优结果；**last.pt**是最后一次结果，用于继续训练

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/results.png" caption="results.png" >}}

## 4.5 预测

找到yolov5项目目录下的**detect.py**，修改几个参数：

1. **'--weights'** 自己的**best.pt**路径
2. **'--source'** 预测源，我直接连了摄像头，所以是0

修改完直接运行开始预测，验证识别的情况

## 4.6 导出其他格式

**best.pt**是pyTorch格式的，我们可以将其转换成其他格式，比如**onnx**格式

找到yolov5项目目录下的**export.py**，修改权重路径和导出格式，直接运行导出

**onnx**是一种中间格式，他可以转成其他几乎所有的格式，也可以直接在装有**onnxruntime**的环境上运行

对于没有GPU的树莓派来说，**onnx**比较能发挥CPU的性能

> 一些好用的工具：
>
> [netron.app 查看导出的onnx结构](https://netron.app/)
>
> [convertmodel.com 可以优化onnx结构 进一步提升推理速度](https://convertmodel.com/)
>
> [编译好的onnxruntime wheel，能直接在树莓派上安装](https://github.com/nknytk/built-onnxruntime-for-raspberrypi-linux/tree/master)

## 4.7 自己优化的目标检测脚本（针对摄像头输入）

根据**detect.py**优化，去除了用不到的代码，增加了一下功能，关键代码如下：

```python
import json
import logging.config
import multiprocessing
import socket

import RPi.GPIO as GPIO
import bluetooth
import serial
import yaml
from serial import SerialException
import contextlib
import logging.config
import math
import os
import platform
import re
import time
from pathlib import Path
from threading import Thread

import cv2
import numpy as np
import onnxruntime
import torch
import torch.nn as nn
import torchvision
from PIL import Image

from ultralytics.utils.plotting import Annotator, colors

uart = None

def detect(source):
    weights = '/home/pi/script/ud/best.onnx'  # model path or triton URL
    # source = 0  # file/dir/URL/glob/screen/0(webcam)
    data = ''  # dataset.yaml path
    imgsz = (640, 640)  # inference size (height, width)
    conf_thres = 0.25  # confidence threshold
    iou_thres = 0.45  # NMS IOU threshold
    max_det = 1000  # maximum detections per image
    device = 'cpu'  # cuda device, i.e. 0 or 0,1,2,3 or cpu
    classes = None  # filter by class: --class 0, or --class 0 2 3
    agnostic_nms = False  # class-agnostic NMS
    augment = False  # augmented inference
    visualize = False  # visualize features
    line_thickness = 3  # bounding box thickness (pixels)
    hide_labels = False  # hide labels
    hide_conf = False  # hide confidences
    half = False  # use FP16 half-precision inference
    dnn = False  # use OpenCV DNN for ONNX inference
    vid_stride = 1  # video frame-rate stride
    pi_mode = True  # 树莓派模式 pi_mode_time检测一次图片 节省资源
    pi_mode_time = 5  # 树莓派模式 检测间隔 单位秒
    show_video = False  # 是否显示视频
    img_dir = '/home/pi/script/ud/exp/'  # 图片保存地址

    source = str(source)

    # save_dir = increment_path(Path(project) / name, exist_ok=exist_ok)

    # Load model
    device = torch.device(device)
    model = DetectMultiBackend(weights, device=device, dnn=dnn, data=data, fp16=half)
    stride, names, pt = model.stride, model.names, False
    imgsz = check_img_size(imgsz, s=stride)  # check image size

    # Dataloader
    # view_img = check_imshow(warn=True)
    dataset = LoadStreams(source, img_size=imgsz, stride=stride, auto=pt, vid_stride=vid_stride)
    bs = len(dataset)

    # Run inference
    model.warmup(imgsz=(1 if pt or model.triton else bs, 3, *imgsz))  # warmup
    seen, windows, dt = 0, [], (Profile(), Profile(), Profile())
    for path, im, im0s, vid_cap, s in dataset:
        with dt[0]:
            im = torch.from_numpy(im).to(model.device)
            im = im.half() if model.fp16 else im.float()  # uint8 to fp16/32
            im /= 255  # 0 - 255 to 0.0 - 1.0
            if len(im.shape) == 3:
                im = im[None]  # expand for batch dim

        # Inference
        with dt[1]:
            pred = model(im, augment=augment, visualize=visualize)

        # NMS
        with dt[2]:
            pred = non_max_suppression(pred, conf_thres, iou_thres, classes, agnostic_nms, max_det=max_det)

        # Second-stage classifier (optional)
        # pred = utils.general.apply_classifier(pred, classifier_model, im, im0s)

        # Process predictions
        for i, det in enumerate(pred):  # per image
            seen += 1
            p, im0, frame = path[i], im0s[i].copy(), dataset.count
            s += f'{i}: '

            p = Path(p)  # to Path
            s += '%gx%g ' % im.shape[2:]  # print string
            annotator = Annotator(im0, line_width=line_thickness, example=str(names))
            if len(det):
                # Rescale boxes from img_size to im0 size
                det[:, :4] = scale_boxes(im.shape[2:], det[:, :4], im0.shape).round()

                # Print results
                for c in det[:, 5].unique():
                    n = (det[:, 5] == c).sum()  # detections per class
                    s += f"{n} {names[int(c)]}{'s' * (n > 1)}, "  # add to string

                # Write results
                for *xyxy, conf, cls in reversed(det):
                    # Add bbox to image
                    c = int(cls)  # integer class
                    label = None if hide_labels else (names[c] if hide_conf else f'{names[c]} {conf:.2f}')
                    annotator.box_label(xyxy, label, color=colors(c, True))

            # Stream results
            if show_video:
                im0 = annotator.result()
                if platform.system() == 'Linux' and p not in windows:
                    windows.append(p)
                    cv2.namedWindow(str(p), cv2.WINDOW_NORMAL | cv2.WINDOW_KEEPRATIO)  # allow window resize (Linux)
                    cv2.resizeWindow(str(p), im0.shape[1], im0.shape[0])
                cv2.imshow(str(p), im0)
                cv2.waitKey(1)  # 1 millisecond

            if len(det):
                # save img
                cv2.imwrite(f'{img_dir}{time.time_ns()}.jpg', im0)
                # Print time (inference-only)
                logger.info(
                    f"{'camera '}{s}{'' if len(det) else '(no detections), '}{dt[1].dt * 1E3:.1f}ms")

        if pi_mode:
            time.sleep(pi_mode_time)
            # logger.info(f'sleep {pi_mode_time}s')

    # Print results
    t = tuple(x.t / seen * 1E3 for x in dt)  # speeds per image
    logger.info(f'Speed: %.1fms pre-process, %.1fms inference, %.1fms NMS per image at shape {(1, 3, *imgsz)}' % t)


class DetectMultiBackend(nn.Module):
    # YOLOv5 MultiBackend class for python inference on various backends
    def __init__(self, weights='', device=torch.device('cpu'), dnn=False, data=None, fp16=False, fuse=True):

        super().__init__()
        w = str(weights[0] if isinstance(weights, list) else weights)
        pt, jit, onnx, xml, engine, coreml, saved_model, pb, tflite, edgetpu, tfjs, paddle, triton = [False, False,
                                                                                                      True, False,
                                                                                                      False, False,
                                                                                                      False, False,
                                                                                                      False, False,
                                                                                                      False, False,
                                                                                                      False]
        nhwc = False
        stride = 32  # default stride
        cuda = torch.cuda.is_available() and device.type != 'cpu'  # use CUDA

        providers = ['CUDAExecutionProvider', 'CPUExecutionProvider'] if cuda else ['CPUExecutionProvider']
        session = onnxruntime.InferenceSession(w, providers=providers)
        output_names = [x.name for x in session.get_outputs()]
        meta = session.get_modelmeta().custom_metadata_map  # metadata
        if 'stride' in meta:
            stride, names = int(meta['stride']), eval(meta['names'])

        self.__dict__.update(locals())  # assign all variables to self

    def forward(self, im, augment=False, visualize=False):
        # YOLOv5 MultiBackend inference
        b, ch, h, w = im.shape  # batch, channel, height, width
        if self.fp16 and im.dtype != torch.float16:
            im = im.half()  # to FP16
        if self.nhwc:
            im = im.permute(0, 2, 3, 1)  # torch BCHW to numpy BHWC shape(1,320,192,3)

        im = im.cpu().numpy()  # torch to numpy
        y = self.session.run(self.output_names, {self.session.get_inputs()[0].name: im})

        if isinstance(y, (list, tuple)):
            return self.from_numpy(y[0]) if len(y) == 1 else [self.from_numpy(x) for x in y]
        else:
            return self.from_numpy(y)

    def from_numpy(self, x):
        return torch.from_numpy(x).to(self.device) if isinstance(x, np.ndarray) else x

    def warmup(self, imgsz=(1, 3, 640, 640)):
        # Warmup model by running inference once
        warmup_types = self.pt, self.jit, self.onnx, self.engine, self.saved_model, self.pb, self.triton
        if any(warmup_types) and (self.device.type != 'cpu' or self.triton):
            im = torch.empty(*imgsz, dtype=torch.half if self.fp16 else torch.float, device=self.device)  # input
            for _ in range(2 if self.jit else 1):  #
                self.forward(im)  # warmup


def check_img_size(imgsz, s=32, floor=0):
    # Verify image size is a multiple of stride s in each dimension
    if isinstance(imgsz, int):  # integer i.e. img_size=640
        new_size = max(make_divisible(imgsz, int(s)), floor)
    else:  # list i.e. img_size=[640, 480]
        imgsz = list(imgsz)  # convert to list if tuple
        new_size = [max(make_divisible(x, int(s)), floor) for x in imgsz]
    return new_size


def make_divisible(x, divisor):
    # Returns nearest x divisible by divisor
    if isinstance(divisor, torch.Tensor):
        divisor = int(divisor.max())  # to int
    return math.ceil(x / divisor) * divisor


def check_imshow(warn=False):
    # Check if environment supports image displays
    try:
        cv2.imshow('test', np.zeros((1, 1, 3)))
        cv2.waitKey(1)
        cv2.destroyAllWindows()
        cv2.waitKey(1)
        return True
    except Exception as e:
        if warn:
            logger.warning(f'WARNING ⚠️ Environment does not support cv2.imshow() or PIL Image.show()\n{e}')
        return False


class LoadStreams:
    # YOLOv5 streamloader, i.e. `python detect.py --source 'rtsp://example.com/media.mp4'  # RTSP, RTMP, HTTP streams`
    def __init__(self, sources='file.streams', img_size=640, stride=32, auto=True, transforms=None, vid_stride=1):
        torch.backends.cudnn.benchmark = True  # faster for fixed-size inference
        self.mode = 'stream'
        self.img_size = img_size
        self.stride = stride
        self.vid_stride = vid_stride  # video frame-rate stride
        sources = Path(sources).read_text().rsplit() if os.path.isfile(sources) else [sources]
        n = len(sources)
        self.sources = [clean_str(x) for x in sources]  # clean source names for later
        self.imgs, self.fps, self.frames, self.threads = [None] * n, [0] * n, [0] * n, [None] * n
        for i, s in enumerate(sources):  # index, source
            # Start thread to read frames from video stream
            st = f'{i + 1}/{n}: {s}... '

            s = eval(s) if s.isnumeric() else s  # i.e. s = '0' local webcam
            cap = cv2.VideoCapture(s)
            cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter.fourcc('M', 'J', 'P', 'G'))
            assert cap.isOpened(), f'{st}Failed to open {s}'
            logger.info(f'camera {s} is open')
            w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
            h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
            fps = cap.get(cv2.CAP_PROP_FPS)  # warning: may return 0 or nan
            self.frames[i] = max(int(cap.get(cv2.CAP_PROP_FRAME_COUNT)), 0) or float('inf')  # infinite stream fallback
            self.fps[i] = max((fps if math.isfinite(fps) else 0) % 100, 0) or 30  # 30 FPS fallback

            _, self.imgs[i] = cap.read()  # guarantee first frame
            self.threads[i] = Thread(target=self.update, args=([i, cap, s]), daemon=True)
            logger.info(f'{st} Success ({self.frames[i]} frames {w}x{h} at {self.fps[i]:.2f} FPS)')
            self.threads[i].start()

        # check for common shapes
        s = np.stack([letterbox(x, img_size, stride=stride, auto=auto)[0].shape for x in self.imgs])
        self.rect = np.unique(s, axis=0).shape[0] == 1  # rect inference if all shapes equal
        self.auto = auto and self.rect
        self.transforms = transforms  # optional
        if not self.rect:
            logger.warning('WARNING ⚠️ Stream shapes differ. For optimal performance supply similarly-shaped streams.')

    def update(self, i, cap, stream):
        # Read stream `i` frames in daemon thread
        n, f = 0, self.frames[i]  # frame number, frame array
        while cap.isOpened() and n < f:
            n += 1
            cap.grab()  # .read() = .grab() followed by .retrieve()
            if n % self.vid_stride == 0:
                success, im = cap.retrieve()
                if success:
                    self.imgs[i] = im
                else:
                    logger.warning('WARNING ⚠️ Video stream unresponsive, please check your IP camera connection.')
                    self.imgs[i] = np.zeros_like(self.imgs[i])
                    cap.open(stream)  # re-open stream if signal was lost
            time.sleep(0.0)  # wait time

    def __iter__(self):
        self.count = -1
        return self

    def __next__(self):
        self.count += 1
        if not all(x.is_alive() for x in self.threads) or cv2.waitKey(1) == ord('q'):  # q to quit
            cv2.destroyAllWindows()
            raise StopIteration

        im0 = self.imgs.copy()
        if self.transforms:
            im = np.stack([self.transforms(x) for x in im0])  # transforms
        else:
            im = np.stack([letterbox(x, self.img_size, stride=self.stride, auto=self.auto)[0] for x in im0])  # resize
            im = im[..., ::-1].transpose((0, 3, 1, 2))  # BGR to RGB, BHWC to BCHW
            im = np.ascontiguousarray(im)  # contiguous

        return self.sources, im, im0, None, ''

    def __len__(self):
        return len(self.sources)  # 1E12 frames = 32 streams at 30 FPS for 30 years


def clean_str(s):
    # Cleans a string by replacing special characters with underscore _
    return re.sub(pattern='[|@#!¡·$€%&()=?¿^*;:,¨´><+]', repl='_', string=s)


def letterbox(im, new_shape=(640, 640), color=(114, 114, 114), auto=True, scaleFill=False, scaleup=True, stride=32):
    # Resize and pad image while meeting stride-multiple constraints
    shape = im.shape[:2]  # current shape [height, width]
    if isinstance(new_shape, int):
        new_shape = (new_shape, new_shape)

    # Scale ratio (new / old)
    r = min(new_shape[0] / shape[0], new_shape[1] / shape[1])
    if not scaleup:  # only scale down, do not scale up (for better val mAP)
        r = min(r, 1.0)

    # Compute padding
    ratio = r, r  # width, height ratios
    new_unpad = int(round(shape[1] * r)), int(round(shape[0] * r))
    dw, dh = new_shape[1] - new_unpad[0], new_shape[0] - new_unpad[1]  # wh padding
    if auto:  # minimum rectangle
        dw, dh = np.mod(dw, stride), np.mod(dh, stride)  # wh padding
    elif scaleFill:  # stretch
        dw, dh = 0.0, 0.0
        new_unpad = (new_shape[1], new_shape[0])
        ratio = new_shape[1] / shape[1], new_shape[0] / shape[0]  # width, height ratios

    dw /= 2  # divide padding into 2 sides
    dh /= 2

    if shape[::-1] != new_unpad:  # resize
        im = cv2.resize(im, new_unpad, interpolation=cv2.INTER_LINEAR)
    top, bottom = int(round(dh - 0.1)), int(round(dh + 0.1))
    left, right = int(round(dw - 0.1)), int(round(dw + 0.1))
    im = cv2.copyMakeBorder(im, top, bottom, left, right, cv2.BORDER_CONSTANT, value=color)  # add border
    return im, ratio, (dw, dh)


class Profile(contextlib.ContextDecorator):
    # YOLOv5 Profile class. Usage: @Profile() decorator or 'with Profile():' context manager
    def __init__(self, t=0.0):
        self.t = t
        self.cuda = torch.cuda.is_available()

    def __enter__(self):
        self.start = self.time()
        return self

    def __exit__(self, type, value, traceback):
        self.dt = self.time() - self.start  # delta-time
        self.t += self.dt  # accumulate dt

    def time(self):
        if self.cuda:
            torch.cuda.synchronize()
        return time.time()


def non_max_suppression(
        prediction,
        conf_thres=0.25,
        iou_thres=0.45,
        classes=None,
        agnostic=False,
        multi_label=False,
        labels=(),
        max_det=300,
        nm=0,  # number of masks
):
    """Non-Maximum Suppression (NMS) on inference results to reject overlapping detections

    Returns:
         list of detections, on (n,6) tensor per image [xyxy, conf, cls]
    """

    # Checks
    assert 0 <= conf_thres <= 1, f'Invalid Confidence threshold {conf_thres}, valid values are between 0.0 and 1.0'
    assert 0 <= iou_thres <= 1, f'Invalid IoU {iou_thres}, valid values are between 0.0 and 1.0'
    if isinstance(prediction, (list, tuple)):  # YOLOv5 model in validation model, output = (inference_out, loss_out)
        prediction = prediction[0]  # select only inference output

    device = prediction.device
    mps = 'mps' in device.type  # Apple MPS
    if mps:  # MPS not fully supported yet, convert tensors to CPU before NMS
        prediction = prediction.cpu()
    bs = prediction.shape[0]  # batch size
    nc = prediction.shape[2] - nm - 5  # number of classes
    xc = prediction[..., 4] > conf_thres  # candidates

    # Settings
    # min_wh = 2  # (pixels) minimum box width and height
    max_wh = 7680  # (pixels) maximum box width and height
    max_nms = 30000  # maximum number of boxes into torchvision.ops.nms()
    time_limit = 0.5 + 0.05 * bs  # seconds to quit after
    redundant = True  # require redundant detections
    multi_label &= nc > 1  # multiple labels per box (adds 0.5ms/img)
    merge = False  # use merge-NMS

    t = time.time()
    mi = 5 + nc  # mask start index
    output = [torch.zeros((0, 6 + nm), device=prediction.device)] * bs
    for xi, x in enumerate(prediction):  # image index, image inference
        # Apply constraints
        # x[((x[..., 2:4] < min_wh) | (x[..., 2:4] > max_wh)).any(1), 4] = 0  # width-height
        x = x[xc[xi]]  # confidence

        # Cat apriori labels if autolabelling
        if labels and len(labels[xi]):
            lb = labels[xi]
            v = torch.zeros((len(lb), nc + nm + 5), device=x.device)
            v[:, :4] = lb[:, 1:5]  # box
            v[:, 4] = 1.0  # conf
            v[range(len(lb)), lb[:, 0].long() + 5] = 1.0  # cls
            x = torch.cat((x, v), 0)

        # If none remain process next image
        if not x.shape[0]:
            continue

        # Compute conf
        x[:, 5:] *= x[:, 4:5]  # conf = obj_conf * cls_conf

        # Box/Mask
        box = xywh2xyxy(x[:, :4])  # center_x, center_y, width, height) to (x1, y1, x2, y2)
        mask = x[:, mi:]  # zero columns if no masks

        # Detections matrix nx6 (xyxy, conf, cls)
        if multi_label:
            i, j = (x[:, 5:mi] > conf_thres).nonzero(as_tuple=False).T
            x = torch.cat((box[i], x[i, 5 + j, None], j[:, None].float(), mask[i]), 1)
        else:  # best class only
            conf, j = x[:, 5:mi].max(1, keepdim=True)
            x = torch.cat((box, conf, j.float(), mask), 1)[conf.view(-1) > conf_thres]

        # Filter by class
        if classes is not None:
            x = x[(x[:, 5:6] == torch.tensor(classes, device=x.device)).any(1)]

        # Apply finite constraint
        # if not torch.isfinite(x).all():
        #     x = x[torch.isfinite(x).all(1)]

        # Check shape
        n = x.shape[0]  # number of boxes
        if not n:  # no boxes
            continue
        x = x[x[:, 4].argsort(descending=True)[:max_nms]]  # sort by confidence and remove excess boxes

        # Batched NMS
        c = x[:, 5:6] * (0 if agnostic else max_wh)  # classes
        boxes, scores = x[:, :4] + c, x[:, 4]  # boxes (offset by class), scores
        i = torchvision.ops.nms(boxes, scores, iou_thres)  # NMS
        i = i[:max_det]  # limit detections
        if merge and (1 < n < 3E3):  # Merge NMS (boxes merged using weighted mean)
            # update boxes as boxes(i,4) = weights(i,n) * boxes(n,4)
            iou = box_iou(boxes[i], boxes) > iou_thres  # iou matrix
            weights = iou * scores[None]  # box weights
            x[i, :4] = torch.mm(weights, x[:, :4]).float() / weights.sum(1, keepdim=True)  # merged boxes
            if redundant:
                i = i[iou.sum(1) > 1]  # require redundancy

        output[xi] = x[i]
        if mps:
            output[xi] = output[xi].to(device)
        if (time.time() - t) > time_limit:
            logger.warning(f'WARNING ⚠️ NMS time limit {time_limit:.3f}s exceeded')
            break  # time limit exceeded

    return output


def box_iou(box1, box2, eps=1e-7):
    (a1, a2), (b1, b2) = box1.unsqueeze(1).chunk(2, 2), box2.unsqueeze(0).chunk(2, 2)
    inter = (torch.min(a2, b2) - torch.max(a1, b1)).clamp(0).prod(2)

    # IoU = inter / (area1 + area2 - inter)
    return inter / ((a2 - a1).prod(2) + (b2 - b1).prod(2) - inter + eps)


def xywh2xyxy(x):
    # Convert nx4 boxes from [x, y, w, h] to [x1, y1, x2, y2] where xy1=top-left, xy2=bottom-right
    y = x.clone() if isinstance(x, torch.Tensor) else np.copy(x)
    y[..., 0] = x[..., 0] - x[..., 2] / 2  # top left x
    y[..., 1] = x[..., 1] - x[..., 3] / 2  # top left y
    y[..., 2] = x[..., 0] + x[..., 2] / 2  # bottom right x
    y[..., 3] = x[..., 1] + x[..., 3] / 2  # bottom right y
    return y


def scale_boxes(img1_shape, boxes, img0_shape, ratio_pad=None):
    # Rescale boxes (xyxy) from img1_shape to img0_shape
    if ratio_pad is None:  # calculate from img0_shape
        gain = min(img1_shape[0] / img0_shape[0], img1_shape[1] / img0_shape[1])  # gain  = old / new
        pad = (img1_shape[1] - img0_shape[1] * gain) / 2, (img1_shape[0] - img0_shape[0] * gain) / 2  # wh padding
    else:
        gain = ratio_pad[0][0]
        pad = ratio_pad[1]

    boxes[..., [0, 2]] -= pad[0]  # x padding
    boxes[..., [1, 3]] -= pad[1]  # y padding
    boxes[..., :4] /= gain
    clip_boxes(boxes, img0_shape)
    return boxes


def clip_boxes(boxes, shape):
    # Clip boxes (xyxy) to image shape (height, width)
    if isinstance(boxes, torch.Tensor):  # faster individually
        boxes[..., 0].clamp_(0, shape[1])  # x1
        boxes[..., 1].clamp_(0, shape[0])  # y1
        boxes[..., 2].clamp_(0, shape[1])  # x2
        boxes[..., 3].clamp_(0, shape[0])  # y2
    else:  # np.array (faster grouped)
        boxes[..., [0, 2]] = boxes[..., [0, 2]].clip(0, shape[1])  # x1, x2
        boxes[..., [1, 3]] = boxes[..., [1, 3]].clip(0, shape[0])  # y1, y2


def xyxy2xywh(x):
    # Convert nx4 boxes from [x1, y1, x2, y2] to [x, y, w, h] where xy1=top-left, xy2=bottom-right
    y = x.clone() if isinstance(x, torch.Tensor) else np.copy(x)
    y[..., 0] = (x[..., 0] + x[..., 2]) / 2  # x center
    y[..., 1] = (x[..., 1] + x[..., 3]) / 2  # y center
    y[..., 2] = x[..., 2] - x[..., 0]  # width
    y[..., 3] = x[..., 3] - x[..., 1]  # height
    return y


def save_one_box(xyxy, im, file=Path('im.jpg'), gain=1.02, pad=10, square=False, BGR=False, save=True):
    if not isinstance(xyxy, torch.Tensor):  # may be list
        xyxy = torch.stack(xyxy)
    b = xyxy2xywh(xyxy.view(-1, 4))  # boxes
    if square:
        b[:, 2:] = b[:, 2:].max(1)[0].unsqueeze(1)  # attempt rectangle to square
    b[:, 2:] = b[:, 2:] * gain + pad  # box wh * gain + pad
    xyxy = xywh2xyxy(b).long()
    clip_boxes(xyxy, im.shape)
    crop = im[int(xyxy[0, 1]):int(xyxy[0, 3]), int(xyxy[0, 0]):int(xyxy[0, 2]), ::(1 if BGR else -1)]
    if save:
        file.parent.mkdir(parents=True, exist_ok=True)  # make directory
        f = str(increment_path(file).with_suffix('.jpg'))
        # cv2.imwrite(f, crop)  # save BGR, https://github.com/ultralytics/yolov5/issues/7007 chroma subsampling issue
        Image.fromarray(crop[..., ::-1]).save(f, quality=95, subsampling=0)  # save RGB
    return crop


def increment_path(path, exist_ok=False, sep='', mkdir=False):
    path = Path(path)  # os-agnostic
    if path.exists() and not exist_ok:
        path, suffix = (path.with_suffix(''), path.suffix) if path.is_file() else (path, '')

        # Method 1
        for n in range(2, 9999):
            p = f'{path}{sep}{n}{suffix}'  # increment path
            if not os.path.exists(p):  #
                break
        path = Path(p)

    if mkdir:
        path.mkdir(parents=True, exist_ok=True)  # make directory

    return path


def is_json(data):
    try:
        json.loads(data)
        return True
    except:
        return False


if __name__ == '__main__':

    t_detect_0 = multiprocessing.Process(target=detect, args=(0,))
    t_detect_0.start()

    t_detect_1 = multiprocessing.Process(target=detect, args=(2,))
    t_detect_1.start()

```

> 2023/9/5 更新多摄像头实现方式

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/detect.png" caption="拍的显示器比较模糊但是也能识别到" >}}

{{< image src="https://blog.porrizx.cc:9004/data/blog-img/minio-blog-pic/pi2/detect-pi.png" caption="在树莓派上运行 每张图片的推理速度在200ms左右输出视频能有5帧左右" >}}

# 5 遇到的问题和解决方法

## 5.1 连接多个摄像头出现问题

连接多个摄像头，用opencv展示视频输出时，一只摄像头一直报`select timeout`

原因是树莓派的CPU/GPU处理能力有限，或者是主流的usb摄像头驱动uvc有一些限制，最终造成图像数据无法很好处理

解决方法：由于opencv使用uvc在树莓派中读取usb摄像机流，而cv2.VideoCapture可能默认为未压缩的流，例如YUYV，因此我们需要将流格式更改为MJPG之类的内容，具体取决于摄像机是否支持该格式。

在终端输入 `v4l2-ctl -d /dev/video0 --list-formats` 查询你的摄像头支持哪一种流

在用opencv获取摄像头内容的时候用下面的代码转一下

```python
capture.set(cv2.CAP_PROP_FOURCC,cv2.VideoWriter.fourcc('M','J','P','G'))
```

> 参考文章 https://blog.csdn.net/nick_young_qu/article/details/104658955

脚本样例

```python
import multiprocessing

import cv2


def camera(num):
    print(num)
    capture = cv2.VideoCapture(num)

    capture.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter.fourcc('M', 'J', 'P', 'G'))

    if capture.isOpened():
        print(f'{num} open')
        capture.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
        capture.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

        while True:
            read_code, frame = capture.read()
            if read_code:
                cv2.imshow(str(num), frame)

            k = cv2.waitKey(30) & 0xff
            if k == 27:  # Esc for quit
                break

    capture.release()


if __name__ == '__main__':
    t_detect_0 = multiprocessing.Process(target=camera, args=(0,))
    t_detect_1 = multiprocessing.Process(target=camera, args=(2,))

    t_detect_0.start()
    t_detect_1.start()

    cv2.destroyAllWindows()

```
