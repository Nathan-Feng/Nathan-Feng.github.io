---
layout:     post
title:      "Android ROM 开发之SDK常用开发适配方法总结 --Base on Android 10.0"
subtitle:   "快速跨平台移植，减少适配工作量"
date:   2021-01-24 14:25:00 +0800
author:     "Nathan"
tags:

    - ROM
    - SDK
既然
---

> 步伐不必太大，只要方向正确就好
>
> 既然想把世间精彩尽收眼底，就必须踏出舒适圈
>
> Life's so short and such glorious images in the world, and such horror as whell, but I want to see it all. So I've moved out of my confort zone.



### 前言

做了ROM开发也有一段时间了，平台SDK方面开发维护方面也多少积累了一些经验，现在分享给大家，希望对大家有帮助

### 解决问题

本文主要解决的问题有：

1. 如何预制apk，jar，so，等所有文件并编译打包到ROM里
2. 如何适配ir/BT键值到ROM里
3. 如何自定义自己的厂家目录，跨芯片移植时省时省力
4. 开发问题定位

### 问题原因

由于Andorid TV产品开发，生命周期要求的比较严格，一年一个大版本，所以就要求各个厂家每年都会适配一个大版本，加上不同的芯片平台都需要适配，所以也产生了跨平台移植本厂定制化的内容。这就要求在跨版本和平台时，尽量减少适配时间，增加开发产品时间才是王道。本文的目的也是给大家一个思路，也是个人的总结，日后查找方便。

### Android 编译系统

我们知道Android编译步骤就三个`source build/envsetup.sh` , `lunch product` , `make -j16` 然后就进入了`/device/manufacturer/device-name/device.mk` 中，本文就从它开始吧。











