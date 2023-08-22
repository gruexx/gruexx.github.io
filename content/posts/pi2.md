---
title: "树莓派（CM4）学习日志记录 【2】 OpenCV Yolov5 摄像头 目标检测"
date: 2023-08-07T12:09:38+08:00
draft: false
description: "树莓派（CM4）学习日志记录 【2-OpenCV Yolov5 摄像头 目标检测】"
tags: [ "树莓派", "Yolov5",  "OpenCV"]
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

{{< image src="/img/pi2/labelImg.png" caption="labelImg 窗口" src_l="/img/pi2/labelImg.png" >}}

> [labelImg 下载地址](https://github.com/HumanSignal/labelImg/releases)
>
> **python最好用3.9版本**

标记完图片会输出一个txt文件，将其放入**labels**文件夹下，原始图片则放到**images**文件夹下

**train**是训练用的，**val**是验证用的，保证图片和txt文件名字对应就行

{{< image src="/img/pi2/coco.png" caption="数据集格式" src_l="/img/pi2/coco.png" >}}

## 4.3 训练

找到yolov5项目目录下的**train.py**，修改几个主要的参数:

1. **'--weights'**  基于该模型训练，在yolov5 github页面下载
2. **'--cfg'** yolov5n.yaml 项目自带的模板 只需要修改文件中类的数量，即nc的值
3. **'--data'** coco128.yaml 项目自带的模板 主要修改数据集的路径和类的名称
4. **'--epochs'** 训练的轮次
5. **'--imgsz'** 图片尺寸 通常是 320或640

{{< image src="/img/pi2/train.png" caption="train.py" src_l="/img/pi2/train.png" >}}

修改完直接运行开始训练

> [Model yolov5n.pt 下载地址](https://github.com/ultralytics/yolov5/releases/download/v7.0/yolov5n.pt)

## 4.4 训练完成

训练完成后在项目的**runs**目录下回输出训练结果

找到**weight**文件夹，这里存放的是pyTorch格式的权重文件

**best.pt**是最优结果；**last.pt**是最后一次结果，用于继续训练

{{< image src="/img/pi2/results.png" caption="results.png" src_l="/img/pi2/results.png" >}}

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
def run(
        weights='best3-sim.onnx',  # model path or triton URL
        source=0,  # file/dir/URL/glob/screen/0(webcam)
        data='',  # dataset.yaml path
        imgsz=(320, 320),  # inference size (height, width)
        conf_thres=0.25,  # confidence threshold
        iou_thres=0.45,  # NMS IOU threshold
        max_det=1000,  # maximum detections per image
        device='cpu',  # cuda device, i.e. 0 or 0,1,2,3 or cpu
        classes=None,  # filter by class: --class 0, or --class 0 2 3
        agnostic_nms=False,  # class-agnostic NMS
        augment=False,  # augmented inference
        visualize=False,  # visualize features
        line_thickness=3,  # bounding box thickness (pixels)
        hide_labels=False,  # hide labels
        hide_conf=False,  # hide confidences
        half=False,  # use FP16 half-precision inference
        dnn=False,  # use OpenCV DNN for ONNX inference
        vid_stride=1,  # video frame-rate stride
        pi_mode=True,  # 树莓派模式 pi_mode_time检测一次图片 节省资源
        pi_mode_time=5,  # 树莓派模式 检测间隔 单位秒
        show_video=False,  # 是否显示视频
):
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
                # Print time (inference-only)
                LOGGER.info(
                    f"{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}{' camera '}{s}{'' if len(det) else '(no detections), '}{dt[1].dt * 1E3:.1f}ms")

        if pi_mode:
            time.sleep(pi_mode_time)
            # LOGGER.info(f'sleep {pi_mode_time}s')

    # Print results
    t = tuple(x.t / seen * 1E3 for x in dt)  # speeds per image
    LOGGER.info(f'Speed: %.1fms pre-process, %.1fms inference, %.1fms NMS per image at shape {(1, 3, *imgsz)}' % t)
```

> [detect.py 完整代码](https://gitee.com/zhou_zz/my_py_script/blob/master/piZxy02/v5/detect.py)

{{< image src="/img/pi2/detect.png" caption="拍的显示器比较模糊 但是也能识别到" src_l="/img/pi2/detect.png" >}}

{{< image src="/img/pi2/detect-pi.png" caption="在树莓派上运行 每张图片的推理速度在200ms左右 输出视频能有5帧左右" src_l="/img/pi2/detect-pi.png" >}}
