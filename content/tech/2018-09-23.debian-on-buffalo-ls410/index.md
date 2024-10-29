---
slug: "debian-on-buffalo-ls410"
title: "在buffalo ls410上安装debian"
keywords: "buffalo,nas,ls410,debian"
description: "Buffalo LS410 NAS 安装Debian并升级到新版，进行了一些补充。"
date: 2018-09-23T21:49:01+08:00
draft: false
---

buffalo ls410 是 Buffalo LinkStation 系列的一款arm based nas，MARVELL Armada 370 CPU，512M 内存。

在ls410上安装Debian，以及一些补充。

> 2020年更新，本文的方法已不适用，有新的安装方法，不依赖原生固件，不需要debootstrap
> https://fire0594.com/tech/debian-on-buffalo-ls410-2020.html

教程参考了多处内容，本段略作介绍

这款单盘位NAS当年销量似乎不高，关于debian的资料很少，而且都没有一条龙式的方案

山下康成的博客 http://www.yamasita.jp/linkstation/category/ls410d/ 应该是关于在 ls410 上安装 debian 最早的探索，不过似乎只有一些零碎的脚本片段，再加上书面日语（尤其是涉及到计算机部分的内容）很难读懂而且机翻也不顶用，我的尝试并不成功

之后尝试过另一篇博客上的 ls421de 安装 debian 的教程 https://tohenk.wordpress.com/2014/11/12/debian-wheezy-on-linkstation-ls421de/ 也不成功，教程中的脚本运行顺利，但debootstrap阶段后reboot就不能启动只有白灯闪了

然后在 github 上的 OpenLinkstation 项目 https://github.com/rogers0/OpenLinkstation 中偶然发现了关于 ls410 的配置代码，但这个项目只支持 ls420，并没有说明支持 ls410，由于 ls410 ls420 ls421 这三款nas都是采用了相同的Armada 370 CPU + 512M 内存配置而且官方固件通用，于是就试了一下。虽然过程中有一些小问题，但最终都解决了，成功为 ls410 安装了debian

过程中一旦系统挂掉就需要tftp boot引导后刷回原固件，教程及工具参考了 http://anp.synology.me/ls420d/

正文开始

## Step.1 安装固件

需要的东西
* ls410官方固件 (http://buffalo.jp/support_ap/support/products/ls400.html)
* tftp boot recovery 工具 (http://www.buffalo-technology.com/userfiles/file/downloads/recovery/TFTP%20Boot%20Recovery%20LS400.exe)
* 安装 NAS Navigator2 (http://buffalo.jp/support_ap/support/products/ls400.html)

首先，要安装官方固件，因为之后会用debootstrap建立debian rootfs，而内核使用的是官方固件中的内核，所以要先让ls410升级到最新的官方固件，最新版本1.81是2015年6月发布的，估计以后也不会再更新了(更新：2017年6月27日buffalo发布了1.83版本固件，1.83未测试能否成功debootstrap)

此步骤适合在一块空硬盘或者已崩溃的系统上安装官方固件，如果你的ls410现在运行正常，可以略过这一步直接在线升级

首先用网线把ls410和你的计算机 直连，不需要通过交换机或者路由器等其他网络设备

把你的计算机网卡设置为静态ip，ip地址为192.168.11.1，子网掩码255.255.255.0，关闭计算机中的其他网络设备如无线网卡，关闭计算机中的防火墙

启动ls410

运行 TFTP Boot.exe（tftp boot recovery工具），当看到 accepting requests.. 时，按住ls410的 FUNCTION按钮（机子前面板唯一的按钮） 2～3 秒，此时ls410的指示灯会变成白灯，稍等片刻窗口中会出现两行内容Client ** Blocks Served，此时ls410已经通过tftp boot启动成功了，如果不成功的话重复以上步骤

运行NAS Navigator，如果询问是否自动修改ip地址和时间，选否

在NAS Navigator查看ls410的ip地址，我这边的情况是ls410都会是一个 169.254.*.* 这种在 dhcp failed 时的ip地址，修改计算机的ip地址确保ls410和计算机处于同一网段，比如ls410的ip此时为169.254.127.15，那我们就把计算机的ip设置为169.254.127.16（子网掩码255.255.0.0）

修改ls410官方固件中的升级配置文件 LSUpdater.ini

```sh
[Flags]
 - VersionCheck = 1
 - NoFormatting = 1
 + VersionCheck = 0
 + NoFormatting = 0
 + [SpecialFlags]
 + Debug = 1
```

运行 LSUpdater.exe （ls410官方固件），在窗口标题栏上 右键 > debug，把 Update 和 Config 中的所有选项都选上

注意Rebuild partition table，会 重建分区表，所有数据都会清空，记得备份好数据

点Update开始升级，过程需要一段时间，可以喝喝茶休息一下

如果一切顺利，固件安装成功后ls410重启完毕（白灯常亮），就可以把ls410通过网线连接到路由器上了
不要忘了把计算网卡的ip地址修改回dhcp自动分配

## Step.2 启用ssh

官方固件默认是关闭ssh的，只能通过一个web面板管理ls410

启用ssh的方法有很多，最简单的就是使用acp commander，无需拆机取硬盘，无需打包固件重刷

只需要一个acp commander程序 (https://www.gry.ch/Java/styled/)

进行这一步前要先到web面板中设置好管理员密码

运行 acp_commander_gui_156.exe 会自动找到局域网中的 ls410，输入管理员密码后测试是否能成功连接

逐行输入下面的代码并执行

```sh
chmod 0755 /etc/init.d/sshd.sh
(echo root;echo root)|passwd
sed -i 's/#Port 22/Port 22/g' /etc/sshd_config
sed -i 's/#Protocol 2/Protocol 2/g' /etc/sshd_config
sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/sshd_config
sed -i 's/#StrictModes yes/StrictModes yes/g' /etc/sshd_config
sed -i 's/\/usr\/lib\/sftp-server/\/usr\/local\/libexec\/sftp-server/g' /etc/sshd_config
sed -i 's/"${SUPPORT_SFTP}" = "0"/"${SUPPORT_SFTP}" = "1"/g' /etc/init.d/sshd.sh
/etc/init.d/sshd.sh restart
```

注意执行代码后的回显，确保每一句都成功执行

最后一句重启ssh服务的代码有很高几率执行失败，再试一次就可以

一切顺利的话，现在就可以ssh登陆到ls410了，用户名和密码都是root

## Step.3 安装debian

在ls410上执行以下代码安装debootstrap

```sh
wget http://ftp.cn.debian.org/debian/pool/main/d/debootstrap/debootstrap_1.0.64~bpo70+1_all.deb
dpkg -i --force-all debootstrap_1.0.64~bpo70+1_all.deb
```

将github上的 OpenLinkstation 项目上传到 ls410 (https://github.com/rogers0/OpenLinkstation)

在ls410上执行以下代码部署debian rootfs到用户数据区(sda5)

```sh
chmod -R 777 OpenLinkstation-master
cd OpenLinkstation-master
vi lib/config
1_debootstrap/build_rootfs_with_buffalo-kernel.sh
reboot
```

config文件头几行内容修改如下

```sh
# default settings
DEBOOTSTRAP=debootstrap
ROOTPW=root
DISTRO=wheezy
NEWHOST=LS410
MIRROR=http://ftp.cn.debian.org/debian
FORCE_BUFFALO_KERNEL=1
SYSTEMD=0
```

debootstrap过程比较长，可以再喝杯茶休息下

ls410重启后，会临时启动到debian环境，指示灯是白灯闪烁但系统确实是启动了的，注意此时千万不要关机/重启或意外断电，否则前功尽弃，只能从头来过

在ls410上执行以下代码安装debian到系统区(sda2)

```sh
mkfs.ext3 /dev/sda2
mount /dev/sda2 /mnt
/OpenLinkstation-master/1_debootstrap/build_rootfs_with_buffalo-kernel.sh debian
reboot
```

重启完后ls410就会启动到完整的debian系统，可以随意操作了

此时ls410的指示灯依旧会保持白灯闪烁，后面板的电源切换开关无效，USB接口也无效，后面会解决这些问题

##  Step.4 升级debian 到 jessie

先移除makedev

```sh
apt-get remove makedev
```

修改debian源

```sh
deb http://ftp.jp.debian.org/debian jessie main
deb-src http://ftp.jp.debian.org/debian jessie main
deb http://security.debian.org/ jessie/updates main
deb-src http://security.debian.org/ jessie/updates main
```

升级

```sh
apt-get update
apt-get upgrade
```

升级到jessie

```sh
apt-get dist-upgrade
```

此时肯定会提示内核版本过低，设置udev为不升级后再重复以上步骤：

```sh
echo udev hold | dpkg --set-selections
```

清理

```sh
apt-get autoremove
apt-get clean
```

## Step.4.1 升级debian 到 stretch

修改debian源

```sh
deb http://ftp.jp.debian.org/debian stretch main
deb-src http://ftp.jp.debian.org/debian stretch main
deb http://security.debian.org/ stretch/updates main
deb-src http://security.debian.org/ stretch/updates main
```

升级

```sh
apt-get update
apt-get upgrade
```

升级到stretch

```sh
apt-get dist-upgrade
```

## Step.5 补充

这一步会解决指示灯、电源开关、USB等各种问题

因为使用的是官方固件中的内核，设备都有被识别到，只是缺少控制程序所以未能正常工作

而且官方固件中的驱动模块都会被完整的复制到debian中，只需insmod即可

首先是时区，默认情况下此时ls410应该是日本时区，会比中国北京时间快1个小时，用下面这条命令重新设置时区：

```sh
dpkg-reconfigure tzdata
```

用ntp自动同步系统时间

```sh
apt-get install ntp
```

自动载入USB驱动：

```sh
vi /etc/rc.local
 + insmod /lib/modules/3.3.4/kernel/fs/ufsd/ufsd.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/usb-common.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/core/usbcore.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/host/ohci-hcd.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/host/ehci-hcd.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/host/xhci-hcd.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/storage/usb-storage.ko
 + insmod /lib/modules/3.3.4/kernel/drivers/usb/class/usblp.ko
```

启动完毕后熄灭LED（LED实在太亮了，亮着还不如灭着）：

```sh
vi /etc/rc.local
 + echo off > /proc/buffalo/gpio/led/power_blink
 + echo off > /proc/buffalo/gpio/led/power
```

用后面板的电源切换开关开关机，创建/power_sw.sh文件（记得chmod +x 加上权限），内容如下：

```sh
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/bin
export PATH
for((i=1;i<=11;i++));do
  cat /proc/buffalo/gpio/switch/power|while read status; do
  if [ $status = "off" ]
    then
      echo on > /proc/buffalo/gpio/led/power
      echo on > /proc/buffalo/gpio/led/power_blink
      /sbin/shutdown -h -P now "Power button switched"
      exit
  fi
  done
  sleep 5
done
```

添加cron任务，这样就会每5秒检查一次电源切换开关的状态，如果处于关闭状态就shutdown：

```sh
crontab -e
*/1 * * * * /power_sw.sh
```

调节风扇转速：

风扇转速由文件 /proc/buffalo/gpio/fan/control 控制，有3档转速：full fast slow 和 stop 停转

通过hddtemp获取硬盘温度并调节风扇转速，高于45度时full，低于45度时slow，创建/fan_speed.sh文件（同样，记得chmod +x 加上权限），内容如下：

```sh
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/bin
export PATH
TEMP=`hddtemp -n /dev/sda`
if [ "$TEMP" -le 45 ]
then
  echo on > /proc/buffalo/gpio/led/power
  sleep 0.5
  echo off > /proc/buffalo/gpio/led/power
  echo slow > /proc/buffalo/gpio/fan/control
else
  echo on > /proc/buffalo/gpio/led/info
  sleep 0.5
  echo off > /proc/buffalo/gpio/led/info
  echo full > /proc/buffalo/gpio/fan/control
fi
```

添加cron任务，每1分钟检测一次硬盘温度并调节风扇转速：

```sh
crontab -e
*/1 * * * * /fan_speed.sh
```

添加硬盘休眠功能：

```sh
apt-get install hdparm
vi /etc/rc.local
 + hdparm -S 242 /dev/sda
```

自动挂载swap和用户数据分区：

```sh
mkdir /mnt/sda6
vi /etc/fstab
 + /dev/sda5   swap   swap   sw   0   0
 + /dev/sda6   /mnt/sda6 xfs defaults,noatime 0 0
```

也可以用uuid挂载：

```sh
+ UUID=***** /mnt/sda6 xfs defaults 0 0
```

mount -a 挂载一下试试，没问题的话，以后数据分区都会被自动挂载到/mnt/sda6
