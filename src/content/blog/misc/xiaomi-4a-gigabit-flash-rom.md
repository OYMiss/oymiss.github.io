---
author: Chongzhuo Yang
title: 小米 4A 千兆版刷机
pubDatetime: 2021-03-25 13:01:00
tags:
  - misc
slug: "xiaomi-4a-gigabit-flash-rom"
description: 小米 4A 千兆版刷机的救砖过程。
---

## 软件刷机

教程可以参考这个 [小米4A千兆版 NO拆机](https://www.right.com.cn/forum/thread-4007071-1-1.html)

但是我一直无法访问进去，telnet 192.168.1.1 一直是拒绝访问。原因是 GitHub 在国内访问不稳定，该下载东西无法下载。

### 预先下载文件

下载好这两个文件，[dropbearStaticMipsel.tar.bz2](https://github.com/acecilia/OpenWRTInvasion/raw/master/script_tools/dropbearStaticMipsel.tar.bz2)、[busybox-mipsel](https://github.com/acecilia/OpenWRTInvasion/raw/master/script_tools/busybox-mipsel)

### 建立文件共享服务器

如果是 macOS：

```bash
# cd 到文件所在目录运行这个，比如 cd ~/Download
sudo python -m SimpleHTTPServer 80
```

如果是 Windows：

据说可以用 [chfs](http://iscute.cn/tar/chfs/2.0/gui-chfs-windows.zip) 来实现类似功能

### 修改 `script.sh` 文件

假设电脑的 ip 是 192.168.31.8，将 OpenWRTInvasion 里面的 `script.sh` 中 `https://github.com/acecilia/OpenWRTInvasion/raw/master/script_tools/` 改成 `http://192.168.31.8/`。

里面代码变成这样。

```
curl -L "http://192.168.31.8/busybox-mipsel" --insecure --output busybox
curl -L "http://192.168.31.8/dropbearStaticMipsel.tar.bz2" --output dropbear.tar.bz2
```

### 执行

然后再执行 `python remote_command_execution_vulnerability.py`，这时候再 telnet 就可以了。

## 编程器刷机（救砖）

不小心路由器成砖了。于是买编程器进行刷机。（之前备份过 eeprom）

拆开路由器（螺丝在贴纸下面），找到 Flash。

> 芯片是分正反的，芯片上右上角有一个凹槽，来指示位置。

<img src="/assets/xiaomi-4a-gigabit-flash-rom/flash.png" style="width: 60%; height: 60%" alt="flash"/>

淘宝买了 CH341A 带夹子的那种。

### 组装编程器和架子

按照下图红框内的图示安装好这个芯片，小米 4A 是 25 系芯片，所以安装在图中下面的槽中。

<img src="/assets/xiaomi-4a-gigabit-flash-rom/programmer.png" style="width: 60%; height: 60%" alt="programmer"/>

圈住的部分对应芯片上的凹槽位置，用来确定正反。

<img src="/assets/xiaomi-4a-gigabit-flash-rom/1position.png" style="width: 60%; height: 60%" alt="1position"/>

按照图示安装架子。

<img src="/assets/xiaomi-4a-gigabit-flash-rom/connect.png" style="width: 60%; height: 60%" alt="connect"/>

### 烧录

用夹子夹住Flash芯片，红线应该对应芯片上凹槽所在的位置。

连接到电脑上（据说要连到 USB3.0接口上）

利用 Windows 虚拟机刷到一半刷机程序未响应，所以直接用命令行进行烧录。

我用的 macOS，如果是 Linux 应该也一样。先去下载 [breed-mt7621-pbr-m1.bin](https://breed.hackpascal.net/breed-mt7621-pbr-m1.bin)。

```bash
# brew install truncate
# brew install flashrom

truncate -s +16670883 breed-mt7621-pbr-m1.bin
flashrom --programmer ch341a_spi -c "W25Q128.V" --write breed-mt7621-pbr-m1.bin
```

运行完命令，breed 就刷好了。救砖成功！

[1] [小米4A千兆版 NO拆机](https://www.right.com.cn/forum/thread-4007071-1-1.html)

[2] [Xiaomi WiFi Router 3G V2](https://forum.openwrt.org/t/xiaomi-wifi-router-3g-v2/42584/46)

[3] [小米路由器4A千兆版（R4A）之编程器刷机](https://www.right.com.cn/forum/thread-3161868-1-1.html)

[4] [小米路由器4A千兆版编程器刷入PandoraBox](https://www.right.com.cn/forum/thread-3908050-1-1.html)
