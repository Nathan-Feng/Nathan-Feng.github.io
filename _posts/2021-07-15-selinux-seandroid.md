---
layout:     post
title:      "SELinux->SEAndroid"
subtitle:   "making your device more security"
date:   2021-07-15 20:42:00 +0800
author:     "Nathan"
tags:

    - SELinux
    - Android

---

> 延长白天的最好方法，就是从夜晚偷几个小时。
>
> The best of all ways to lengthen our days is to steal some hours from the night



### 前言

做Android平台SDK开发，在Android 8.0版本之后面临不少系统权限问题，经过这几年的开发和适配以及平时的探索，我在系统权限和安全方面学习和掌握了不少这方面的知识。

本文基于Android 11.0, 主要介绍下SELinux和SEAndroid，本文在阅读参考了国内外不少大牛的资料基础上，加上平时的实战总结出来的，希望对大家有所帮助。

### 安全系统的重要性

对于安卓系统，比如一台手机，如果系统不安全，不稳定，那么很可能就会出现如下问题：

* 系统经常崩溃，影响用户体验
* 对于各种bug的第三方应用，影响系统使用
* 对于恶意应用，会入侵入侵用户的私有数据，串改数据等
* 系统被网络黑客入侵与控制

可以想象，对于生产厂家和用户来说，如上的情况是多么的糟糕，公司倒闭，各种被唾弃都能可能出现。。。

所以构建一个**坚固**的系统多么的重要

### Linux标准安全机制介绍：

众所周知，Android底层是Linux内核，那么关于安全部分，肯定离不开Linux，首先简单介绍下Linux的权限机制。

用户user: uid gid gids
进程process: uid gid gids，其继承于所属用户，子进程继承父进程
文件系统file: uid gid 以及相对应rwx权限

### DAC介绍

占楼

### Android沙箱介绍

占楼


### SELinux介绍

占楼

### MAC介绍

占楼

### SEAndroid介绍

占楼

### SEAndroid实战

占楼

### MLS介绍

占楼

### Code & Doc

主要代码和文档汇总

谷歌在线文档：[https://source.android.google.cn/security/selinux](https://source.android.google.cn/security/selinux)


源码路径 ：/system/sepolicy

网友1：[https://blog.csdn.net/innost/article/details/19299937/](https://blog.csdn.net/innost/article/details/19299937/)

网友2：[https://blog.csdn.net/luoshengyang/article/details/35392905](https://blog.csdn.net/luoshengyang/article/details/35392905)

全文 完！
