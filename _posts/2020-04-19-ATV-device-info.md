---
layout:     post
title:      "AndroidTV获取设备常用信息"
subtitle:   "AndroidTV device info demo based on Java"
date:   2020-04-19 17:00:00 +0800
author:     "Nathan"
tags:
    - Tools

---

> 人生中那些有意义的事，大多都是有代价的
>
> Very few things that are worthwhile in life come without a cost
>

## 前言

在Android开发过程中，我会经常使用串口或者adb命令直接来调试以及查看设备的相关信息，比如查看网络的IP、MAC啦，cat一些参数啊，或者getprop等都比较方便。但是对于用户或者非开发者来说就不能这么操作了，因此针对一些常用的参数获取，开发了一个demo，并封装了常用的调用接口，满足常用的功能获取与展示。

### 接口总结

有些设备信息需要系统签名权限，而有些则SDK中自带了，所以根据权限不同，本文分为两大接口，**系统级API**和**非系统级API**，我这里总结了大部分常用的功能接口，如下：

### 非系统级权限接口(`NonSystemDeviceInfoApi`)

* `getCpuNumber()` //查看CPU 个数

* `getCpuInfo()` //获取所有CPU相关的信息 （`cat /proc/cpuinfo`）返回结果如下图

  ![cpuinfo](/img/device-info/cat-cpuinfo.jpg)

* `getFreeMemInfo()` //获取当前时刻的剩余内存空间

* `getTotalMemInfo()` //获取总内存的空间

* `getTotalExternalFlashSize()` //获取Sdcard总空间

* `getAvailableExternalFlashSize()` //获取Sdcard可用空间

* `exeShellCmd(String cmd)` //执行shell脚本的接口，例如：ls 等常用linux以及andorid命令

* `getTotalNetworkRxBytes()` //获取当前有线网卡和wifi的总流量之和的数据，

* `getProp()` //在shell下执行getprop后返回的数据

  (**注意**：调用此接口返回的数据总数，会比签名后调用的数据少一些，因为有些属性在普通权限下无法获取)

  ![cpuinfo](/img/device-info/shell-getprop.jpg)
  
* 增加新接口`String[] getSystemLogcatPids()` 通过执行shell 命令拿到目前`logcat`进程的所有pid信息

* 

### 系统级权限接口(`SystemDeviceInfoApi`)

* `getMaxCpuFreq()` //查看CPU的最大频率

* `getMinCpuFreq()` //查看CPU的最小频率

  ![cpuinfo](/img/device-info/cpu-freq.png)

* `getSerialNo()` //获取设备序列号`ro.serial.no`，经过测试验证，必须要签名才能获取

* `getEthernetMacAddress()` //获取有线网卡MAC地址 

* `getWifiMacAddress()` //获取wifi网卡MAC地址

* `getEthIpAddress()` //获取有线网卡IP地址

* `getWifiIpAddress()` //获取Wifi的IP地址

  ![cpuinfo](/img/device-info/wifi-mac-ip.png)

* `getGpuFreqRange()` //获取GPU的频率范围(根据平台不同，会有差异，目前以RTK1319平台为测试)

* `String[][] getAllNetData()` // 获取有线网卡和Wifi的各自流量，可以计算实时网速(通过`cat /proc/self/net/dev`命令获取相关参数)

* 增加获取cpu使用率的接口`getCurCpuRate`

* 增加系统唤醒接口`forceSystemWakeUp`

* 增加系统待机接口`forceSystemStandby`

* 增加系统重启接口`forceSystemReboot`

* 增加判断当前是否待机接口`isSystemStandby`

* 增加恢复出厂接口`factoryReset`

* 增加获取system进程pid的接口`getSystemProcessPid`

* 增加kill system进程pid的接口`killSystemProcessPid`

* 增加获取root进程pid的接口`getRootProcessPid`

* 增加kill root进程pid的接口`killRootProcessPid`

* 增加静默卸载app接口`uninstallPkg`

### 新增蓝牙接口

`BluetoothDeviceInfoApi.java`

	- 获取绑定列表
	- 获取设备蓝牙名字
	- 获取设备蓝牙MAC
	- 使能蓝牙
	- 重置蓝牙

### 新增网络相关接口

`NetworkDeviceInfoApi.java`

	- 获取有线相关信息`getEthernetInfo`
	- 获取wifi相关信息 `getWifiItemInfo`
	- 设置有线或者wifi相关信息`setNetworkStaticIpInfo`
	- 打开wifi，关闭wifi，重置wifi等

新增存储接口

`StorageDeviceInfoApi.java`

### 通用BuildInfo接口

Android的SDK提供了`android.os.Build`用于通用的信息，大家可以直接查看源码，再次不多解释

### Demo结构说明

**demo地址**：[https://github.com/Nathan-Feng/AndroidTVDeviceInfo](https://github.com/Nathan-Feng/AndroidTVDeviceInfo)

目录结构如下：

![cpuinfo](/img/device-info/demo-tree.png)

主要包括API接口层，后台服务UI层，接口实现层

附上效果图：

* 启动页面，需要用户手动来手动授权启动浮窗位置(`Settings->Apps->Special app access->Display over other apps->your app->Open(Allowed)`)

![cpuinfo](/img/device-info/setting-allowed.png)

* 主页面

![cpuinfo](/img/device-info/main-activity.png)

* 所有效果

![cpuinfo](/img/device-info/open-all.png)

如果觉得有用，欢迎给个**Star**，Thank you 