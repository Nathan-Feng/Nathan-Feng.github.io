---
layout:     post
title:      "Android System App 执行 root shell cmd 解决方案 --Base on Android 10.0"
subtitle:   "黑色产业链:System App->jni->hidl->root shell "
date:   2021-01-21 21:12:00 +0800
author:     "Nathan"
tags:

    - Tools
    - API
    - Selinux
    - HIDL

---

> 步伐不必太大，只要方向正确就好
>
> The steps you take don't need to be big. They just need to  take you in the right direction.
>



### 前言

最近项目有个需求，需要通过app来调用执行sytem/bin或者vendor/bin中的可执行程序，也就是要执行root shell 命令。通过研究分析对比和测试，最终找到了一条可行途径，并且能符合谷歌CTS认证需求，对于后续平台开发和扩展都有重要意义，因此总结分享出来，希望可以帮助更多ROM开发的童鞋。话不多说，进入正题。

### 前置条件

本文主要涉及如下知识内容(有了解过就好，不要求深入)，如果不了解的，可以自己先补一下：

 - Android app开发(java语言)
 - Android ROM源码编译(Make, Android.mk, Android.bp等)
 - NDK开发（主要是jni, 简单的C++,C）
 - Android binder通信
 - Linux Shell 命令
 - Android HIDL 方面的知识
 - Selinux方面的知识

### Android App的权限

我们知道，Android ROM中，app分为几个等级：

	untrusted_app 表示第三方app，用户自安装的app，没有Android平台签名，没有system权限
	platform_app 表示有android平台签名，没有system权限 (Apps signed with the platform key)
	system_app 表示有android平台签名和system权限(在AndroidManifest.xml标签中加入属性：android:sharedUserId="android.uid.system")(Apps that run with the system UID)

相应的权限等级，理论上：untrusted_app < platform_app < system_app,  我们今天讨论的就是放在system/app下面的system_app，使其在USER模式下拥有root权限。

### 执行Shell方法

Android app开发中(以java为例) ，一般执行shell命令的简单方法：`Runtime.getRuntime().exec(isRooted ? "su" : "sh");`调用一些简单的命令如logcat等，不过由于System app的权限问题，不能以su命令来执行,而都是shell权限执行，导致很多root下的可执行程序无法正常使用。



