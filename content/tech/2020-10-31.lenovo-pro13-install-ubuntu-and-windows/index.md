---
slug: "lenovo-pro13-install-ubuntu-and-windows"
title: "联想小新Pro13装系统，从Ubuntu到Windows"
keywords: "小新pro13,linux,ubuntu,重装系统"
description: "联想小新Pro13 2020核显款装系统，原本打算装ubuntu，但会有一些错误，于是换回了windows"
date: 2020-10-31T17:10:50+08:00
draft: false
---

联想小新Pro13 2020核显款，原本是打算装Ubuntu使用，但是装完后dmesg里面会有错误

虽然部分错误可以修复，且机器运行正常使用也正常，但还是不放心，最后老老实实装回Windows

这里记录一下过程

## Ubuntu 20.04

#### 桌面环境选择

**XFCE**

一直都是用的 Xubuntu，也就是XFCE桌面环境的Ubuntu

但是在13寸2.5K的高DPI屏幕，XFCE的表现实在不尽人意，缩放只有100%和200%，前者看不清，后者像老年机

然后因为XFCE没有 client side decorations，缩放对 Xfwm4 标题栏无效，对部分程序无效比如 virtualbox

目前XFCE是4.14，要等4.16才会有 fractional scaling 和 client side decorations

**KDE**

又试了Kubuntu，也就是KDE桌面环境的Ubuntu，可以 125%、150%、175% 这样缩放了

但是会有一些小问题，有一部分缩放是无效的，还有一部分位置会偏移很多比如输入法候选框

**Gnome**

最后选择了原始Ubuntu，Gnome桌面环境，缩放工作完美

#### 处理内核错误

装完之后机器运行正常，但是dmesg一敲一片红，错误出现在三个地方：硬盘、网卡、声卡

硬盘和声卡可以解决，网卡不行，升级内核也无解

网卡是AC9462，开关网卡和休眠都能产生错误，但是还是能正常工作的

**硬盘错误**：

```sh
PCIe Bus Error: severity=Corrected, type=Physical Layer
```

解决办法：

添加 “pcie_aspm=off”

```sh
sudo vi /etc/default/grub
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"   =>   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_aspm=off"
sudo update-grub
```

参考：

> https://askubuntu.com/questions/863150/pcie-bus-error-severity-corrected-type-physical-layer-id-00e5receiver-id

**声卡错误**：

```sh
sof-audio-pci   error: no reply expected, received 0x0
sof-audio-pci   firmware boot complete
```

解决办法：

添加“options snd-intel-dspcfg dsp_driver=1”

```sh
sudo vi /etc/modprobe.d/alsa-base.conf
+ options snd-intel-dspcfg dsp_driver=1
```

参考：

> https://bbs.archlinux.org/viewtopic.php?id=253494
> https://bugzilla.kernel.org/show_bug.cgi?id=205959



除了通过手动调整硬盘和声卡之外，也可以直接 升级内核：

```sh
sudo apt install linux-headers-5.8.0-25-generic linux-image-5.8.0-25-generic linux-modules-5.8.0-25-generic linux-modules-extra-5.8.0-25-generic
```

**在LTS版本上升级内核并不妥当，建议直接安装20.10的非LTS版本，默认就是5.8内核**

现在除了网卡的错误之外，dmesg里面不会有其他问题了，我使用了一天下来也是没有出现问题的

虽然用起来没问题，但dmesg里的错误还是让人觉得不舒服，还是老老实实装Windows吧

## Windows 10

进PE，用WinNTSetup部署wim镜像，一气呵成。

原以为会一切顺利，可进了安装页面就傻眼了，网卡没驱动不识别就算了，触控板居然也不识别。

刚才在PE里面触控板都正常工作的，我直接就？？？

虽然可以用键盘或者外接鼠标完成安装，然后再安装驱动，但这显然很不优雅

&nbsp;

于是考虑在部署wim镜像的时候直接注入驱动

到联想官网把驱动下载下来，一看都是exe打包好的驱动又是一阵？？？

记得以前联想提供的驱动不是这样的，没办法只能自己提取了

&nbsp;

联想的驱动安装包是用 Inno Setup 打包的，可以用 innounp 提取文件:

```sh
innounp -x Audio-SPSI02AF0A2T.exe -dAudio-SPSI02AF0A2T
```

这样就能把 Audio-SPSI02AF0A2T.exe 释放到 Audio-SPSI02AF0A2T 文件夹，其他文件以此类推

所有驱动都提取出来后，就可以在 WinNTSetup 部署镜像的时候添加驱动了

这样之后系统里面驱动就全有了，不用手动装驱动了。

参考：

> http://innounp.sourceforge.net/
