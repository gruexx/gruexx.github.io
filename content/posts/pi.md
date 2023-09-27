---
title: "树莓派（CM4·CM4IO）学习日志记录 【1】 基础"
subtitle: 树莓派基础功能
date: 2023-08-02T11:50:05+08:00
lastmod: 2023-09-27T16:40:38+08:00
draft: false
description: "树莓派镜像烧录串口UART蓝牙"
tags: [ "树莓派", "镜像烧录", "串口UART", "蓝牙" ]
categories: [ "学习笔记" ]
featuredImage: "https://blog.porrizx.cc:7103/data/blog-img/pi/pinOut.png"
featuredImagePreview: "https://blog.porrizx.cc:7103/data/blog-img/pi/pinOut.png"
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

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/gpio.png" caption="gpio readall 返回信息" >}}

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

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/uart0.png" caption="uart0串口信息 GPIO14输出GPIO15输入" >}}

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

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/tty.png" caption="ttyAMA1~4都开启了" >}}

### 4.2.2 查看串口对应的GPIO口

```shell
sudo cat /sys/kernel/debug/pinctrl/fe200000.gpio-pinctrl-bcm2711/pinmux-pins
```

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/tty-pins.png" caption="串口和针脚的对应关系很明显了" >}}

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

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/J8-gpio.png" caption="图片更明显" >}}

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

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/aurt-test.png" caption="执行结果" >}}

**其他串口的测试同上**

# 5 蓝牙通讯

## 5.1 和手机蓝牙配对

```shell
# 进入蓝牙配对界面
bluetoothctl

# 扫描蓝牙设备
scan on

# 找到自己手机的蓝牙地址并配对
power on
agent on
pair 40:F9:46:50:32:69
```

## 5.2 与手机通讯

要与手机通讯必须先修改一些配置

1. `sudo usermod -G bluetooth -a pi` 添加用户到蓝牙组

2. `sudo nano /etc/systemd/system/dbus-org.bluez.service` 修改蓝牙配置文件

   {{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/bluez.png" caption="蓝牙配置文件修改后" >}}

   > ExecStart=/usr/libexec/bluetooth/bluetoothd -C
   >
   > ExecStartPost=/usr/bin/sdptool add SP

3. `sudo reboot` 重启树莓派

4. `hciconfig -a` 查看蓝牙名称

5. `sudo systemctl status bluetooth` 查看蓝牙服务状态

6. `sudo hciconfig hci0 piscan` 设置蓝牙可被发现

7. `sudo rfcomm watch hci0` 等待蓝牙设备连接

使用手机上的蓝牙调试的app连接树莓派的蓝牙

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/pi/rf.png" caption="连接成功" >}}

连接成功后可以看到多出来一个串口 **/dev/rfcomm0**，通过这个串口和手机通信

写个python脚本向串口发送数据和接收数据

```python
import serial

# 连接串口
uart = serial.Serial(port="/dev/rfcomm0", baudrate=9600)

# 发送数据
send = uart.write("Hello World rfcomm0\n".encode("gbk"))

# 接收数据
while True:
    rev = uart.read(24).decode('utf-8')
    print(rev)
```

## 5.3 蓝牙天线配置为外置天线

编辑 /boot/config.txt 文件

`sudo nano /boot/config.txt`

在文件末尾加入一行配置：

`dtparam=ant2`

> [如何将树莓派 CM4 的 WiFi 天线配置为外置天线](https://shumeipai.nxez.com/2021/12/23/how-to-configure-raspberry-pi-cm4-wifi-use-the-external-antenna.html)

## 5.4 设置蓝牙一直可见

```shell
# 进入蓝牙控制台
sudo bluetoothctl

# 开启蓝牙可见
discoverable on 

# 设置蓝牙一直可见
discoverable-timeout 0
```

## 5.5 将rfcomm配置成服务

新建文件 `rfcomm.service`

```service
[Unit]
Description=RFCOMM service
After=bluetooth.service
Requires=bluetooth.service

[Service]
Restart=on-failure
ExecStart=/usr/bin/rfcomm watch hci0

[Install]
WantedBy=multi-user.target
```

放在`/etc/systemd/system/`目录下，完成后执行`systemctl daemon-reload`和 `systemctl enable rfcomm`

```shell
# 启动
systemctl start rfcomm

# 停止
systemctl stop rfcomm

# 重启
systemctl restart rfcomm

# 状态
systemctl status rfcomm
```

## 5.6 自动确认蓝牙的配对请求设置

配置`bluetoothctl`的代理模式为`NoInputNoOutput` 这将允许树莓派自动确认来自其他设备的配对请求，而无需用户手动确认

以下是具体的步骤：

```shell
# 进入蓝牙控制台
bluetoothctl

# 代理模式设置
agent off

agent NoInputNoOutput

default-agent
```

设置完后，蓝牙配对时只需要在连接端点击配对确认
