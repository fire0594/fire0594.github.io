---
slug: "openwrt-dumb-ap-and-wifi-roaming"
title: "OpenWrt路由器设置AP和Wi-Fi漫游"
keywords: "openwrt,AP,wifi漫游,802.11k,802.11v,802.11r"
description: "多台OpenWrt路由器设置为AP，并使用802.11 k/v/r协议进行Wi-Fi漫游"
date: 2025-03-12T02:22:00+08:00
draft: false
---

几台路由器都是OpenWrt 24.10.0系统，设置为AP后进行Wi-Fi漫游。

## 1.AP设置（Dumb AP）

关键

* 主路由器的LAN口 应该连接 AP路由器的LAN口 而不是 WAN口
* AP路由器的IP可以是静态IP或者由主路由器的DHCP分配
* AP路由器不提供 DHCP 服务，由主路由器负责

疑惑

> AP路由器的dnsmasq和防火墙等是否要关闭？
> 旧版文档说这一步是为了节省资源，但新版说实际好处不大，且配置访客网络时应该启用他们，所以什么都不做就好

> AP路由器的WAN口/WAN6口是否要删除？
> 旧版文档说这一步是可选步骤，新版则未作任何说明，所以这应该依旧是个可选步骤

### `先将AP路由器断开网络，只有电脑通过网线连接到其LAN口，然后开始设置`

### 1.1  AP路由器的LAN口设置为静态地址（static IP）

进入AP路由器的OpenWrt管理页面: `网络 -> 接口 -> LAN -> 编辑 -> 常规设置`
设置IP为静态地址，要和主路由器在同一个子网，并且不能在主路由器DHCP地址池内。
主路由器IP地址为192.168.1.1，DHCP地址池为192.168.1.100-192.168.1.249。
所以AP路由器的IP应该是`192.168.1.2-192.168.1.99`，为每个AP指定不同的IP，192.168.1.2、192.168.1.3、192.168.1.4，以此类推。

### 1.2  AP路由器的LAN口设置网关

进入AP路由器的OpenWrt管理页面: `网络 -> 接口 -> LAN -> 编辑 -> 常规设置`
设置网关为主路由器的IP地址`192.168.1.1`

### 1.3  AP路由器的LAN口设置DNS

进入AP路由器的OpenWrt管理页面: `网络 -> 接口 -> LAN -> 编辑 -> 高级设置`
设置DNS服务器为主路由器的IP地址`192.168.1.1`

这一步也可以不设置，客户端会通过DHCP从主路由器获取DNS服务器

### 1.4  AP路由器的LAN口关闭DHCP服务

进入AP路由器的OpenWrt管理页面: `网络 -> 接口 -> LAN -> 编辑 -> DHCP服务器`

`常规设置`里勾选`忽略此接口`
`IPv6设置`里禁用`RA服务`、`DHCPv6服务`、`NDP代理`

### `到这里AP就可以工作了，现在可以用网线连接主路由器的LAN口和AP路由器的LAN口`

### 1.5  AP路由器LED配置（可选）

路由器一般都有LED指示灯，OpenWrt默认配置的LED在WAN口有数据传输时会亮起并闪烁。
现在路由器作为AP，网线连接的是LAN口，需要修改一下LED配置

进入AP路由器的OpenWrt管理页面: `系统 -> LED配置`
默认有个名称为 `internet` 的配置，编辑配置，把设备从WAN改为连接主路由器的LAN口即可。

## 2.AP漫游

关键

* OpenWrt自带的软件包可能是wpad-basic-*，仅支持802.11r和802.11w，需要更换为完整的wpad-*才能支持全部协议
* 启用并配置协议

疑惑

> 802.11 K/V/R 要启用哪些协议？
> OpenWrt的Wi-Fi漫游文档里面说明如下
> 802.11r（“快速过渡”）减少了客户端在漫游到不同BSSID时建立安全连接所需的时间。
> 802.11k（“无线电资源测量”）允许单个BSSID向Wi-Fi客户端提供ESS中包含的其他BSSID和频率列表。这减少了每个客户端需要花费的时间来寻找替代的“更好”SSID，因为它不再需要扫描所有频率。
> 802.11v（“无线网络管理”）这个标准包括“网络辅助漫游”，BSSID可以推荐客户端可以漫游到的替代BSSID。
> 所以K/V/R 3个协议都启用的话体验应该会更好

> 在OpenWrt上启用802.11 K/V/R协议需要哪些软件包？
> OpenWrt的802.11s 无线Mesh文档里面说明如下
> The wpad-basic-* versions only have 802.11r and 802.11w support.
> The wpad-mesh-* versions only have 802.11r/w and 802.11s support.
> The wpad-* are the full version of wpad and have 802.11k/v/r/w and 802.11s support.
> 所以要同时启用K/V/R 3个协议的话需要wpad-*，在源里面有3个
> wpad-mbedtls, wpad-openssl, wpad-wolfssl

### 2.1  更换为完整的wpad包

我的路由器自带的是wpad-basic-mbedtls，更换为wpad-mbedtls
在线或者离线都行，最好用SSH操作

```sh
opkg update && opkg install wpad-mbedtls --download-only && opkg remove wpad-basic-mbedtls && opkg install wpad-mbedtls --cache . && rm *.ipk
```

```sh
opkg remove wpad-basic-mbedtls && opkg install /tmp/wpad-mbedtls_*.ipk
```

现在需要重启一下路由器，一些新增的配置项才会在管理页面出现

### 2.2  WIFI设置

进入AP路由器的OpenWrt管理页面: `网络 -> 无线 -> **** -> 编辑 -> 接口配置`
每个AP路由器的SSID、加密方式、密码都应该相同

### 2.3  启用和配置协议

进入AP路由器的OpenWrt管理页面: `网络 -> 无线 -> **** -> 编辑 -> 接口配置 -> WLAN漫游`

勾选 `802.11r 快速切换`
`NAS ID`  放空，会自动生成
`移动域`  放空使用OpenWrt的默认值，或者所有AP设置为同一数值
`重关联截止时间`  设置为20000，20000是思科的默认数值，OpenWrt默认的1000可能有兼容性问题
`FT协议`  选择FT over the air，FT over DS可能有兼容性问题
`本地生成PMK`  加密方式是WPA3就不要勾选，如果加密方式是WPA2勾不勾选都行，不勾选比较好

勾选 `802.11k RRM`
`邻居报告` 和 `信标报告`  默认会勾选

`802.11v` 只有下面几个配置项
`时间公告` 和 `时间区域` 优化性能
`WNM 睡眠模式`  勾选，优化性能，同时减少频谱占用，减少干扰
`WNM 睡眠模式修复`  不要勾选，表示不使用加密密钥保护睡眠模式下的管理帧
`BSS 过渡`  勾选，优化接入点之间的过渡，优化体验
`代理 ARP`  不要勾选

### 2.4  检查漫游是否成功

手机连接wifi后，在各个AP之间走一走
进入AP路由器的OpenWrt管理页面: `状态 -> 系统日志`

`daemon.notice hostapd: phy1-ap0: AP-STA-DISCONNECTED **:**:**:**:**:**`
上面这条日志是设备断开AP

`daemon.notice hostapd: phy*-ap*: AP-STA-CONNECTED **:**:**:**:**:** auth_alg=ft`
上面这条日志是设备连接AP，并且使用的是FT协议，而不是一般的4次握手认证，漫游成功

#### 参考

> OpenWrt设置为AP（当前版本，英文）
> https://openwrt.org/docs/guide-user/network/wifi/wifiextenders/bridgedap

> OpenWrt设置为AP（历史版本，中文）
> https://openwrt.org/zh/docs/guide-user/network/wifi/dumbap

> OpenWrt Wi-Fi漫游
> https://openwrt.org/zh/docs/guide-user/network/wifi/roaming
> https://openwrt.org/docs/guide-user/network/wifi/roaming

> OpenWrt 802.11s 无线Mesh
> https://openwrt.org/docs/guide-user/network/wifi/mesh/802-11s
