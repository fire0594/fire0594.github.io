---
slug: "debian-on-iomega-hmnhd-ce"
title: "在iomega Hmnhd Ce上安装debian"
keywords: "iomega,nas,debian"
description: "iomega很多年前的一款NAS，安装Debian并升级到新版，对原文进行了一些补充。"
date: 2017-09-17T21:29:56+08:00
draft: false
---

全名Iomega Home Media Network Hard Drive Cloud Edition，简称Iomega hmnhd ce。
Iomega很久以前的一款单盘位NAS，Oxford NAS 7820 cpu，256M 内存。
在Iomega hmnhd ce上安装Debian wheezy并升级至Debian jessie，并对原教程进行一些补充。

---

## 原文

Howto: Install Debian wheezy on a empty hard drive for Iomega Home Media Network Hard Drive, Cloud Edition:

**Warning:**
By doing anything that follows, you could:
* Loose all data on you device
* Void your warranty
* or any combination of the above. So don't proceed unless you know what you're doing and you mean it.

**Prerequisites:**
You will need
* A blank, or to-be-blanked SATA HDD.
* A linux system (or vm, or live cd/usb) in order to use the commands, scripts & files.
* Your linux system already installed parted and gzip.

**Procedure:**
1. Attach the SATA HDD to the machine/vm you will use for this process.
2. Power up the machine/vm and normal boot up;
3. Recognize your HDD's divice name, For example, here will be used /dev/sdX or sdX
4. Download boot sectors data file BootSectors.tgz and save to your machine/vm's /tmp directory;
5. Extract BootSectors.tgz to /tmp/boot;

command:
```sh
mkdir /tmp/boot
tar -C /tmp/boot -zxf BootSectors.tgz
```

Create new partition table and filesystem on HDD(default rootfs partition size:8GB);

command:
```sh
cd /tmp/boot
./mkgptdisk.sh /dev/sdX #[if you want msdos part, command: ./mkdosdisk.sh /dev/sdX]
```

command:
```sh
parted /dev/sdX mkpart primary 8623489024B 100%
mkfs.ext3 /dev/sdX2
```

Write boot sectors data to HDD;

command:
```sh
cd /tmp/boot
./flushsd.sh /dev/sdX
```

Download rootfs file(wheezy.20140112.tgz) and save to your machine/vm's /tmp directory;

Mount HDD's first partition and extract rootfs to it;

command:
```sh
mkdir /tmp/mydisk
mount /dev/sdX1 /tmp/mydisk
cd /tmp
tar -C /tmp/mydisk -zvxf wheezy.20140112.tgz
sync
umount /dev/sdX1
```

Shut down the machine/vm and remove HDD;

Attach the drive to your Iomega HMNHDCE;

Apply power to the device and power up, Your device will normal boot to the Debian system.

Note:
* This Debian system already installed openssh and start the service at bootup, username and password both are root;
* In This kernel, button device use as a gpio keyboard, power button as KEY_POWER, copy button as KEY_SLEEP and reset button as KEY_SUSPEND;
* You can install acpid or input-event-daemon(https://github.com/gandro/input-event-daemon) to define this button's action;
* You may need regenerate ssh service's key:

command:
```sh
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
```

Kown issue:
* Each boot, Ethernet device doesn't bring up to normal at first, in this rootfs /etc/network/interfaces file define a pre-up segment to slove this issue;
* If you install other ethernet device management tools to change ethernet device's settings, please notes this segment must keeped.
* "pre-up ifconfig eth0 hw ether 00:d0:b8:19:40:d7; ifconfig eth0 up ; ifconfig eth0 down ;"
* Ethernet device's macaddress defined in /etc/network/interfaces file pre-up segment, you can modify it. ubootenv's macaddress define is void for kernel.

> 原文出处：https://forum.nas-central.org/viewtopic.php?f=269&t=13953

---

## 中文

在一块空的（或准备被清空的）硬盘中安装debian，进行下文中的操作，硬盘中的数据会丢失，产品会失去保修

需要的东西：
* 一块空的（或准备被清空的）硬盘
* 一台运行linux的计算机（虚拟机或者live cd也可以）
* 计算机中已经安装了parted和gzip

下载BootSectors.tgz（http://downloads.iomega.nas-central.org/Users/olderzeus/kernel3.12.6%20&%20Debian%20wheezy/BootSectors.tgz）
下载wheezy.20140112.tgz（http://downloads.iomega.nas-central.org/Users/olderzeus/kernel3.12.6%20&%20Debian%20wheezy/wheezy.20140112.tgz）
将BootSectors.tgz和wheezy.20140112.tgz保存到/tmp目录中

> 原网站已失效，文件备份至 https://drive.google.com/drive/folders/1k1qSeQS8ySBHKzaBM8KHBVvB9-3mw3fv?usp=sharing

将硬盘安装到计算机上并启动，确保硬盘被正确识别，记住硬盘的设备名，一般是/dev/sd*这样的，我这里是/dev/sdd

创建/tmp/boot目录并解压BootSectors.tgz到目录下

```sh
mkdir /tmp/boot
tar -C /tmp/boot -zxf BootSectors.tgz
```

为硬盘创建分区表和第一个分区
以下命令为gpt格式分区表，第一个分区为8G

```sh
cd /tmp/boot
./mkgptdisk.sh /dev/sd*
```

如果想用mbr格式分区表，使用以下命令，第一个分区为20G

```sh
cd /tmp/boot
./mkdosdisk.sh /dev/sd*
```

为硬盘创建第二个分区
这一步可以省略，等NAS启动后再进行也可以

```sh
parted /dev/sd* mkpart primary 8623489024B 100%
mkfs.ext3 /dev/sd*2
```

将启动信息写入硬盘

之前的硬盘分区操作会在硬盘头部保留一块32M的未分区块，这一步将会向这32M写入信息，包括u-boot、uimage、initrd等

```sh
cd /tmp/boot
./flushsd.sh /dev/sd*
```

挂载第一个分区并写入rootfs

```sh
mkdir /tmp/mydisk
mount /dev/sd*1 /tmp/mydisk
cd /tmp
tar -C /tmp/mydisk -zvxf wheezy.20140112.tgz
sync
```

原rootfs中设置的debian源位于美国，dns也是8.8.8.8，这里修改一下

```sh
vi /tmp/mydisk/etc/resolv.conf
- nameserver 8.8.8.8
- nameserver 8.8.4.4
+ nameserver 114.114.114.114
+ nameserver 223.5.5.5
vi /tmp/mydisk/etc/apt/sources.list
- deb http://ftp.us.debian.org/debian wheezy main non-free contrib
+ deb http://ftp.cn.debian.org/debian wheezy main non-free contrib
umount /dev/sd*1
```

现在已经可以将硬盘安装到Iomega hmnhd ce并启动到Debian wheezy了
系统已经安装了openssh，用户名和密码都是root

首次启动后
ssh登陆到NAS后

```sh
apt-get update
apt-get upgrade
```

为硬盘创建第二个分区（如果前面没进行这一步的话）

```sh
parted /dev/sd* mkpart primary 8623489024B 100%
mkfs.ext4 /dev/sda2
```

配置fstab自动挂载第二个分区

```sh
mkdir /mnt/sda2
vi /etc/fstab
+ /dev/sda2 /mnt/sda2 ext4 defaults,noatime 0 0
```

也可以用blkid命令获取uuid，使用uuid挂载

```sh
+ UUID=***** /mnt/sda2 ext4 defaults,noatime 0 0
```

挂载一下试试

```sh
mount -a
```

没问题的话，以后每次启动，第二个分区都会被自动挂载到/mnt/sda2

重新生成ssh密钥（可选）

```sh
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
```

修改网卡MAC地址（可选）

NAS每次启动，网卡都需要一个pre-up，位于文件/etc/network/interfaces，网卡MAC地址也是在这里进行设置

如果之前有为NAS在路由器中设置DCHP固定地址而绑定了MAC地址（一般BT用户做端口映射都需要），改成和原来一样的MAC地址就不用到路由器中再次设置

```sh
vi /etc/network/interfaces
```

---

## 升级到Debian jessie

之前一直用着wheezy没啥问题，不过Debian 9 (stretch)即将冻结，马上就要取代jessie成为下一个stable了，还是升级一下吧

修改源配置文件

```sh
vi /etc/apt/sources.list
- deb http://ftp.cn.debian.org/debian wheezy main non-free contrib
+ deb http://ftp.cn.debian.org/debian jessie main non-free contrib
+ deb-src http://ftp.cn.debian.org/debian jessie main non-free contrib
+ deb http://security.debian.org/ jessie/updates main non-free contrib
+ deb-src http://security.debian.org/ jessie/updates main non-free contrib
```

升级

```sh
apt-get update
apt-get upgrade
```

设置udev为不升级（否者下一步会提示内核版本过低）

```sh
echo udev hold | dpkg --set-selections
```

升级到jessie

```sh
apt-get dist-upgrade
```

清理

```sh
apt-get autoremove
```

## 补充

**LED控制**

NAS启动过程中，LED4常亮，LED3闪烁
NAS启动完成后，LED3、LED4常亮
LED3和LED4是白色的LED而且亮度很高，太瞎眼，系统启动完成后就让他们熄灭吧

/sys/class/leds目录下有5个目录分别对应4个LED
为什么5个目录对应4个LED？因为LED3是双色LED

对应关系为：
* status:misc:otb - LED1.QT_蓝色
* status:hdd:blue - LED2.硬盘_蓝色
* status:system:ng - LED3.系统_红色
* status:system:ok - LED3.系统_白色
* status:misc:power - LED4.电源_白色

这5个目录下的brightness文件控制着LED的亮度

系统启动完成后让LED3、LED4熄灭，在rc.local中添加两行命令

```sh
vi /etc/rc.local
+ echo 0 > /sys/class/leds/status:misc:power/brightness
+ echo 0 > /sys/class/leds/status:system:ok/brightness
```

**按键**

Iomega hmnhd ce有三个按键，这里内核将他们识别为GPIO按键

对应关系为：
* 电源键，后面板大按键 - KEY_POWER
* QT键，前面板唯一的按键 - KEY_SLEEP
* 复位键，后面板小孔 - KEY_SUSPEND

安装acpid

```sh
apt-get update
apt-get install acpid
```

安装完acpid后电源键会自动配置为关机功能

配置其他按键的功能，参考/etc/acpi/events/powerbtn-acpi-support和/etc/acpi/powerbtn-acpi-support.sh
