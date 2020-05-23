---
layout:     post
title:      "利用树莓派搭建无线 Time Machine 硬盘"
subtitle:   "a setup for wireless Time Machine hard disk based on Raspberry pi"
date:       2017-08-03
author:     "Moubei"
tags:
    - Raspberry
    - Time Machine
    - MacBook
---

Time Machine 是 MacBook 最有力的助手和保障，可以在命悬一线的时候力挽狂澜。但是呢，插插硬盘很麻烦也容易忘记，经常一周两周都没有机会同步一次。这里介绍一种简单便捷的利用树莓派打造无线 Time Machine 的方法。

## 所需材料
* MacBook
* Raspberry 一台（我用的是 Raspberry Pi）
* 容量1TB 及以上的移动硬盘

## 移动硬盘的准备
移动硬盘几乎并不需要有特别的要求，理论上，任何格式包括`HFS+`，`NTFS`和`ext4`都支持。我们也不需要特地去格式化它，已经存在的文件只是占用了空间而已。

但是，目前还没有找到继承以往的 Time Machine 到新的无线 Time Machine 的方法。所以新的树莓派 Time Machine 将会从头开始备份。

一切就绪就插到树莓派上吧。

## Netatalk 和 Amahi 的安装和配置

> Netatalk is an OpenSource software package, that can be used to turn a \*NIX machine into an extremely high-performance and reliable file server for Macintosh computers.—— Introduction to Netatalk

根据 [Netatalk官网](http://netatalk.sourceforge.net)介绍，Netatalk 是一个开源的软件，它的最大作用就是能让一个基于 Unix 的系统（比如树莓派）成为一个 Macintosh 电脑的文件服务器，其中的关键就是对 [AFP](https://en.wikipedia.org/wiki/Apple_Filing_Protocol)（Apple Filing Protocol）的支持。

#### 在树莓派上安装 Netatalk
```shell
apt-get install netatalk
```

#### 设置共享文件夹
```shell
nano /etc/netatalk/AppleVolumes.default
```

在该配置文件的最下面，是要共享的文件夹，默认包含`~/`，也就是用户 Home 文件夹，可以选择留下或者注释掉。

在此之后加上要分享的其他文件夹，如果作为普通的文件夹，形如
```shell
/media/pi/xiaomao                       "xiaomao"
```
引号内的是共享时所使用的名字。

对于一会儿要作为 Time Machine 的目录需要在后面加上 options:tm ，形如
```shell
/media/pi/xiaomao/TimeMachine           "Time Machine on Raspberry" options:tm 
```

最终效果如下
![](/assets/img/in-post/post-time-machine-on-raspberry/1.png)

#### 在树莓派上安装和配置 Avahi
> Avahi is a system which enables programs to publish and discover services and hosts running on a local network. ——— Avahi’s Wikipedia

[Avahi](https://www.avahi.org)是一个用来在网络中寻找可以与之连接的主机，打印机以及可共享文件的系统。它的功能类似于[zeroconf](https://en.wikipedia.org/wiki/Zero-configuration_networking)网络以及苹果所实现的 Bonjour。
```shell
apt-get install avahi-daemon
```

安装后进行配置，创建文件
```shell
nano /etc/avahi/services/afpd.service
```

往其写入
```shell
<service-group\>
<name replace-wildcards="yes"\>%h\</name\>
<service\>
<type\>_afpovertcp._tcp\</type\>
<port\>548\</port\>
</service\>
<service\>
<type\>_device-info._tcp\</type\>
<port\>0\</port\>
<txt-record\>model=Xserve\</txt-record\>
</service\>
</service-group\>
```
完成配置。

#### 重启 `Netatalk` 和 `Avahi`，使其配置生效
```shell
service netatalk restart
service avahi-daemon restart
```

#### 获取内网 IP 地址
可以通过路由器配置页面获取，或者简单通过一行代码得知
```shell
ifconfig
```
![](/assets/img/in-post/post-time-machine-on-raspberry/2.png)

如图中`inet addr:10.8.44.128`所见，我的树莓派的内网 IP 地址为`10.8.44.128`。

## 从 MacBook 连接配置好的树莓派
#### 右键点击 Finder，选择连接服务器，填入树莓派服务器地址，注意通信协议是 `afp`。
![](/assets/img/in-post/post-time-machine-on-raspberry/3.png)

#### 点击连接，登录即可，然后可以看到之前共享的所以文件夹，选择当时的 Time Machine 文件夹。
![](/assets/img/in-post/post-time-machine-on-raspberry/4.png)

#### 输入此Shell命令使得 MacBook 能够使用网络硬盘作为 Time Machine 的目标地址。
```shell
defaults write com.apple.systempreferences TMShowUnsupportedNetworkVolumes 1
```

#### 进入 Time Machine 设置页面，开启，并选择之前共享的 Time Machine 文件夹作为目标硬盘。

#### 点击立即备份，是否已经工作了呢？
![](/assets/img/in-post/post-time-machine-on-raspberry/5.png)

## 结果测试
由于树莓派本身就不包含支持 USB 3.0 的接口，所以树莓派和移动硬盘之间的读写速度理论上也只有40MBps，更何况Raspberry Pi的网卡只有100M，所以理论传输值仅用11MBps。再加上它的USB 和网卡共享带宽，因此它的理论值还会进一步的下降。

经测试，传输速度仅为300~400kBps，可以说是比较不堪，特别是完成第一次备份的时候。或许在后面的不断的增量备份中，能提高可用性。



封面图片来源于 [pixabay](https://pixabay.com/en/hard-drive-technology-computers-1265259/)