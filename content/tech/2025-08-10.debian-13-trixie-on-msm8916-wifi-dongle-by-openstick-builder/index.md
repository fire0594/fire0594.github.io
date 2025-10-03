---
slug: "debian-13-trixie-on-msm8916-wifi-dongle-by-openstick-builder"
title: "随身wifi刷入Debian13(Trixie)"
keywords: "msm8916,snapdragon 410,wifi dongle,debian 13 trixie,openstick builder"
description: "用OpenStick-Builder为msm8916(高通骁龙410)CPU的随身wifi构建并刷入Debian13(Trixie)系统"
date: 2025-10-03T12:52:00+08:00
draft: false
---

3年前的随身wifi，msm8916(高通骁龙410)的CPU，512MB内存+4G硬盘

# 1.构建系统 Debian13 (Trixie)

## 1.1 准备工作

github上有很多fork，这里使用的是最初的原项目
> https://github.com/kinsamanka/OpenStick-Builder

先复制项目到本地
```sh
git clone --recurse-submodules https://github.com/kinsamanka/OpenStick-Builder.git
cd OpenStick-Builder/
```

debian13的libconfig包版本有变化，需要修改一下
```sh
vi scripts/setup.sh
-    libconfig9 \
+    libconfig11 \
```

## 1.2 开始构建

可以`sudo ./build.sh`一步构建，但是为了方便排查过程中的问题，分步操作更好

安装依赖
`sudo scripts/install_deps.sh`

构建 hyp 和 lk2nd
`sudo scripts/build_hyp_aboot.sh`

从高通固件中提取bootloader，在emmc中创建新分区表
`sudo scripts/extract_fw.sh`

debootstrap创建rootfs (这一步会从Debian源下载文件，最好使用科技加速)
`sudo scripts/debootstrap.sh`

构建gadget-tools
`sudo scripts/build_gt.sh`

生成镜像文件
`sudo scripts/build_images.sh`

生成的文件位于项目的files文件夹

# 2.安装EDL和fastboot

后续要用到EDL(高通9008)读写固件，使用这个项目
> https://github.com/bkerler/edl

不建议使用其他项目的EDL
上一步构建系统使用的OpenStick-Builder，其内核来自postmarketOS
postmarketOS对EDL的说明中要求使用这个EDL项目，参考下面这个链接
> https://wiki.postmarketos.org/wiki/Zhihe_series_LTE_dongles_(generic-zhihe)#How_to_enter_flash_mode

项目文档没有说，但是文件`autoinstall.sh`代码内有注释要求GCC版本 >= 14，注意检查系统上GCC的版本

先复制项目到本地，并安装python依赖
```sh
git clone https://github.com/bkerler/edl
cd edl
git submodule update --init --recursive
pip3 install -r requirements.txt
```

安装依赖，这一步同时也会安装fastboot
```sh
sudo apt install adb fastboot python3-dev python3-pip liblzma-dev git
```

文档里有一步`sudo apt purge modemmanager`，会完全删除`modemmanager`
感觉这一步没有必要，但没有去实验

停用并禁止`modemmanager`
```sh
sudo systemctl stop ModemManager
sudo systemctl disable ModemManager
```

安装
```sh
sudo ./autoinstall.sh
```

# 3.刷机

## 3.1 提取原固件分区

备份原固件的这些分区，后续任会用到:
* fsc.bin
* fsg.bin
* modem.bin
* modemst1.bin
* modemst2.bin
* persist.bin
* sec.bin

由于之前已经刷过Openstick，先使用`edl wf backup.bin`回到原固件

然后用下面的代码提取分区
```sh
for n in fsc fsg modem modemst1 modemst2 persist sec; do
    edl r ${n} ${n}.bin
done
```

目前发现`ufi001`和`ufi003`板型的这些分区可以通用
在一块`ufi001`上提取的分区，在另一块`其他品牌的ufi001`和`ufi003`上正常工作

## 3.2 写入新固件

刷入aboot
`edl w aboot aboot.mbn`

重启到fastboot
```sh
edl e boot
edl reset
```

刷入`步骤1`构建的系统
```sh
fastboot flash partition gpt_both0.bin
fastboot flash aboot aboot.mbn
fastboot flash hyp hyp.mbn
fastboot flash rpm rpm.mbn
fastboot flash sbl1 sbl1.mbn
fastboot flash tz tz.mbn
fastboot flash boot boot.bin
fastboot flash rootfs rootfs.bin
```

刷入`步骤3.1`提取的原固件分区
```sh
for n in fsc fsg modem modemst1 modemst2 persist sec; do
    fastboot flash ${n} ${n}.bin
done
```

刷机结束，重启设备
`fastboot reboot`

# 4.系统配置

## 4.1 SSH登录

默认有1个wifi热点，SSID为`Openstick`，密码为`openstick`
连接这个热点或者插入USB(NDIS有线网卡)，SSH登录`192.168.5.1`，用户名`user`，密码`1`
之前 Debian 12 时热点是正常的，但是 Debian 13 的热点在设备启动后没有自动启用，原因不明，不过不影响后续使用

## 4.2 板型调整

项目支持多种板型，默认设置是UZ801
需要根据板型调整配置，目前发现如果配置不正确LED不能正常工作

使用下面这行代码
`sudo sed -i 's/yiming-uz801v3/<BOARD>/' /boot/extlinux/extlinux.conf`

其中的<BOARD>根据板型替换:
| <BOARD>      | 板型    |
| ------------ | ------ |
| thwc-uf896   | UF896  |
| thwc-ufi001c | UFIxxx |
| jz01-45-v33  | JZxxx  |
| fy-mf800     | MF800  |

## 4.3 配置LED

默认启动完成后红灯常亮，修改为蓝灯常亮
默认没有启用rc-local

先`sudo vi /etc/rc.local`文件内容如下
```sh
#!/bin/bash

echo 0 > /sys/class/leds/red:power/brightness
echo 1 > /sys/class/leds/blue:wan/brightness

exit 0
```

添加运行权限
```sh
sudo chmod +x /etc/rc.local

```

再`sudo vi /etc/systemd/system/rc-local.service`创建rc-local服务，文件内容如下
```sh
[Unit]
Description=/etc/rc.local Compatibility
ConditionFileIsExecutable=/etc/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

最后重新载入并启用rc-local
```sh
sudo systemctl daemon-reload
sudo systemctl enable rc-local
sudo systemctl start rc-local
```

## 4.4 配置wifi

和openstick原项目相同，使用`nmtui`进行配置，参考下面的链接
> https://www.kancloud.cn/handsomehacker/openstick/2637559

## 4.5 修改hostname

```sh
sudo hostnamectl set-hostname openstick-debian-001
sudo vi /etc/hosts
```
