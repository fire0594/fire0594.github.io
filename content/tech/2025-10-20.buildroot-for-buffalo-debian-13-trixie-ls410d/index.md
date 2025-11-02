---
slug: "buildroot-for-buffalo-linkstation-ls410d"
title: "使用 buildroot 为 Buffalo Linkstation LS410D 构建系统"
keywords: "buffalo linkstation ls410d,armada 370,buildroot for buffalo"
description: "用 Buildroot_for_Buffalo 为 Buffalo Linkstation LS410D 构建并安装系统"
date: 2025-11-02T12:27:00+08:00
draft: false
---


# 构建

使用这个项目
> https://github.com/1000001101000/Buildroot_for_Buffalo


**构建过程中要注意:**

需要联网下载文件，最好用科技加速

构建完成的文件夹总容量会达到41GB，注意磁盘空间

需要用到一些库，按提示安装即可

```sh
apt install wget curl git
apt install make gcc g++
apt install libncurses-dev
apt install unzip rsync bc
```

部分库缺少时会报错，但不会提示少了哪个，手动排查后安装上即可

```sh
apt install gcc-multilib g++-multilib
apt install libssl-dev
```

## 下面是构建流程

准备构建

```sh
git clone https://github.com/1000001101000/Buildroot_for_Buffalo
cd Buildroot_for_Buffalo
wget https://buildroot.org/downloads/buildroot-2025.08.1.tar.xz
tar xf buildroot-2025.08.1.tar.xz
cp configs/armada370_defconfig buildroot-2025.08.1/.config
cd buildroot-2025.08.1/
```

配置 Device Tree

```sh
make menuconfig
```

进入 `kernel -> In-tree Device Tree Source file names`
输入 `armada-370-linkstation-ls410d`

推荐配置

进入 `System configuration -> System hostname` 修改主机名

默认 `System configuration -> Enable root login with password` 已允许root密码登陆
进入 `System configuration -> Root password` 设置一下root密码

开始构建

```sh
make -j$(nproc) || make -j$(nproc)
```

构建好的磁盘镜像位于 Buildroot_for_Buffalo/buildroot-2025.08.1/output/images

# 安装

参考项目wiki
> https://github.com/1000001101000/Buildroot_for_Buffalo/wiki/marvell-armada370

1. 直接写入整个磁盘镜像
```sh
sudo dd if=disk.img of=/dev/sda bs=4k
```

2. 手动分区后写入 (我使用这个方法)
```sh
sudo dd if=boot.img of=/dev/sda1 bs=4k
sudo dd if=rootfs.ext4 of=/dev/sda2 bs=4k
```
