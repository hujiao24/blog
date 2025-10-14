---
categories:
- 默认分类
date: '2025-10-02 11:03:24+08:00'
description: ''
draft: false
image: ''
slug: miwifi
cover: /archives/miwifi/7wux3i.png
tags:
- wifi
title: win10下小米随身wifi驱动的安装
---

上古时期的小米随身 wifi，在 win10 下安装了官方的驱动程序，启动的时候，会尝试创建 wifi

但是小米随身 wifi 的驱动最初是为 win 7/8 开发的，官方并没有针对 win 10/11 的完整适配，最终创建 wifi 失败

网上提供了两种方式

（1）定位到 `C:\Program Files (x86)\XiaoMi\MiWiFi\drivers\Win81x64\netr28ux.inf` 文件，然后右键安装，部分机器可能会生效

（2）小米随身 wifi 是基于联发科的无线芯片制造的，因此可以指定使用联发科的驱动

事实上国内大部分随身 wifi 的无线芯片都是联发科，那么使用其他品牌的随身 wifi 无线驱动，理论上也是可行的

这里以指定联发科的驱动为例，完整的操作步骤见下图

（1）设备列表中选定小米随身 wifi，设备可能在 “其他设备” 中，也可能在 “网络适配器” 中

![](/archives/miwifi/7wux3i.png)

（2）右键 “更新驱动程序”

![](/archives/miwifi/7q21h8.png)

（3）浏览我的电脑查找驱动程序

![](/archives/miwifi/r92jtp.png)

![](/archives/miwifi/fmpfjg.png)

（4）双击 “显示所有设备”，然后稍等一会，待列表刷新出来

![](/archives/miwifi/ufs1ly.png)

![](/archives/miwifi/g5v5jy.png)

（5）在显示的厂商列表中，选择 `MediaTe, Inc.`，这是联发科品牌，是 mifiwi 的无线芯片提供商，右侧选择 `802.11n USB 无线 LAN 卡`， 然后点击下一步安装

![](/archives/miwifi/9ttk7e.png)

（6）安装完毕后，网络中就能正常看到 WLAN 的图标了，右键连接 wifi 后，有线连接就会断开，经验证 wifi 可以正常使用

![](/archives/miwifi/6a2gzl.png)