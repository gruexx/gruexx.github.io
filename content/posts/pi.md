---
title: "树莓派（CM4/CM4IO）学习日志记录 【1-基础】"
date: 2023-08-02T11:50:05+08:00
draft: false
description: "树莓派（CM4）学习日志记录 【1】"
tags: [ "树莓派" ]
categories: [ "学习" ]
featuredImage: "/img/pi/pinOut.png"
featuredImagePreview: "/img/pi/pinOut.png"
---

# 1 镜像烧录（EMMC版本）

## 1.1 下载镜像

这里我选择不带桌面的版本 Raspberry Pi OS Lite

[镜像下载](https://www.raspberrypi.com/software/operating-systems/)

## 1.2 进入烧录模式

- 首先需要将CM4 boot短接，然后通过Micro USB /Type C 接口转USB接口连接电脑，再连接电源
- 此时，电脑**设备管理器**的**通用串行总线设备**中会识别出一个BCMxxx Boot的设备
- 运行**rpiboot（引导加载程序）**，等待运行结束，在我的电脑上面会出现一个U盘的盘符
- 使用**SDFormatter**格式化EMMC
- 烧录的准备工作就完成了！！！

## 1.3 烧写镜像

- （不推荐）使用**Win32DiskImager**烧写镜像，需要将下载的 .xz 结尾的镜像文件解压得到 .img 结尾的镜像文件，然后进行烧写
- （推荐）使用官方的**树莓派镜像烧录器 Raspberry Pi Imager**进行烧录，可以直接使用 .xz
  文件烧录，还能设置主机名、开启ssh服务、ssh登录用户名密码、wifi、语音
- 烧录完毕之后，在我的电脑会识别出一个U盘的盘符
- 如果是使用**Win32DiskImager**烧写镜像，还需要开启下ssh服务和设置登录的用户名密码：
- 在根目录下新建一个名称为**ssh**的文件（没有后缀名）来开启ssh服务
- 创建 userconf.txt文件，该文件内输入用户名和密码的密文，以下例子，用户名为pi，密码为raspberry的密文

```
pi:$6$4ilokQRQxmURT.py$aJWBQ5yniJJPwV3CKawYJcnSK5JZGhrVZYF3K4omRUFv6KL0MysEH7F4NZRMNMcYF.U3xsQvWrx7ZL2GKxuv.1
```

> 下载 [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

## 1.4 烧录完成并重启

- 烧录完毕断开电源，断开和电脑的连接线，将BOOT断开，重新上电
- 连接网线，然后使用shell工具连接树莓派

# 2 系统软件安装

- `sudo passwd root` 设置root密码

## 2.1 安装wiringpi

```shell
# 选择要保存的目录
cd download 
 
# 下载deb包
wget https://github.com/WiringPi/WiringPi/releases/download/2.61-1/wiringpi-2.61-1-armhf.deb
 
# 安装deb包
sudo dpkg -i wiringpi-latest.deb

# 查看版本
gpio -v

# 查看gpio信息
gpio readall

# 查看板子详细信息
pinout
```

{{< image src="/img/pi/gpio.png" caption="gpio readall 返回信息" src_l="/img/pi/gpio.png" >}}

# 3 python 脚本编写

## 3.1 RPi.GPIO 模块使用

### 3.1.1 导入模块

```python
import RPi.GPIO as GPIO
```

### 3.1.2 针脚编号

有两种编号可以用来定位针脚（参考上方的图 "gpio readall 返回信息"）：

第一种方式是使用 BOARD 编号系统，使用Physical编号，即物理的编号位置。

第二种方式是使用 BCM 编号，使用BCM的编号。

```python
GPIO.setmode(GPIO.BOARD)
```

或者

```python
GPIO.setmode(GPIO.BCM)
```

### 3.1.3 设置输入输出

```python
# 配置为输入的通道
GPIO.setup(pin_num, GPIO.IN)

# 配置为输出的通道
GPIO.setup(pin_num, GPIO.OUT)
```

> pin_num 是针脚的编号

### 3.1.4 针脚电压

```python
# 查看针脚电压状态
GPIO.input(pin_num)

# 设置针脚输出高电压
GPIO.output(pin_num, GPIO.HIGH)

# 设置针脚输出低电压
GPIO.output(pin_num, GPIO.LOW)
```

> 苹果有个app  **PiHelper** 能直接在手机上修改针脚的输入输出和高低电压，很方便

### 3.1.5 清理

脚本结束后的清理，恢复所有使用过的通道状态为输入，该操作仅会清理脚本使用过的 GPIO 通道。

```python 
GPIO.cleanup()
```

# 4 UART开启

CM4有6个UART，其中默认的开启的串口**UART0**和**UART1**都是**GPIO14**和**GPIO15**
可以通过shell命令查看

```shell
dtoverlay -h uart0
```

{{< image src="/img/pi/uart0.png" caption="uart0串口信息 GPIO14输出 GPIO15输入" src_l="/img/pi/uart0.png" >}}

现在要开启剩余的4个UART串口

## 4.1 修改config.txt

```shell
vi /boot/config.txt
```

在文本末尾加上

```
dtoverlay=uart2
dtoverlay=uart3
dtoverlay=uart4
dtoverlay=uart5
```

添加完重启

```shell
reboot
```

## 4.2 查看UART串口信息

### 4.2.1 查看串口是否开启

```shell
dmesg | grep tty
```

{{< image src="/img/pi/tty.png" caption="ttyAMA1~4都开启了" src_l="/img/pi/tty.png" >}}

### 4.2.2 查看串口对应的GPIO口

```shell
sudo cat /sys/kernel/debug/pinctrl/fe200000.gpio-pinctrl-bcm2711/pinmux-pins
```

{{< image src="/img/pi/tty-pins.png" caption="串口和针脚的对应关系很明显了" src_l="/img/pi/tty-pins.png" >}}

> **可以得出各 UART 串口与 GPIO 对应关系：**
>
> GPIO14 = TXD0 -> ttyAMA0
>
> GPIO0 = TXD2 -> ttyAMA1
>
> GPIO4 = TXD3 -> ttyAMA2
>
> GPIO8 = TXD4 -> ttyAMA3
>
> GPIO12 = TXD5 -> ttyAMA4
>
> ————————————
>
> GPIO15 = RXD0 -> ttyAMA0
>
> GPIO1 = RXD2 -> ttyAMA1
>
> GPIO5 = RXD3 -> ttyAMA2
>
> GPIO9 = RXD4 -> ttyAMA3
>
> GPIO13 = RXD5 -> ttyAMA4

{{< image src="/img/pi/J8-gpio.png" caption="图片更明显" src_l="/img/pi/J8-gpio.png" >}}

### 4.2.3 测试串口是否连通

要测试 **ttyAMA1** 是否通，首先短接 **GPIO0** 和 **GPIO1** ，然后重启树莓派

```shell
# 查看串口波特率
stty -F /dev/ttyAMA1
```

编写python脚本测试发送和接收消息

```python
import serial

uart = serial.Serial(port="/dev/ttyAMA1", baudrate=9600)

send = uart.write("Hello World\n".encode("gbk"))
print(send)

rev = uart.readline()
print(rev)
```

在树莓派上执行

```shell
python aurt-test.py
```

{{< image src="/img/pi/aurt-test.png" caption="执行结果" src_l="/img/pi/aurt-test.png">}}

**其他串口的测试同上**

# 5 蓝牙通讯

## 5.1 和手机蓝牙连接

```shell
# 进入蓝牙配对界面
bluetoothctl

# 扫描蓝牙设备
scan on

# 找到自己手机的蓝牙地址并配对连接
power on
agent on
pair 40:F9:46:50:32:69
```
## 5.2 与手机通讯


