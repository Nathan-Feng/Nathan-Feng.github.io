---
layout:     post
title:      "Android A/B系统升级"
subtitle:   "Android A/B system updates"
date:   2021-05-15 15:33:00 +0800
author:     "Nathan"
tags:

    - 升级
    - Android

---

> 人生短暂，世界很大，我想留下些回忆
>
> Life is short. The world is wide , and I want ot make some memories.



### 前言

随着安卓大版本更新，公司项目中Android 11的SDK，芯片厂家把升级方式从以前的`recovery`升级方式，更新成了`A/B分区升级`的方式，最近在看一些相关文档，并且跑了下sample，进行一些基本功能测试与接口封装。同时也总结了一些文档，特分享出来，供大家参考。

本文主要介绍基本原理和应用层接口调用，已经开发中一些常用辅助命令。不会介绍具体底层代码逻辑(主要是代码太多，比较复杂o(╥﹏╥)o)

### Recovery升级：

对于`recovery`升级原理和介绍，网上的文档说的都比较多，谷歌官网入口：[https://source.android.google.cn/devices/tech/ota/nonab](https://source.android.google.cn/devices/tech/ota/nonab)

这里我就简单说下大致内容

* 首先是将升级包下载或者预置到到 cache 或 userdata 分区
* 其中升级包里是包含可执行二进制文件 META-INF/com/google/android/update-binary，这个文件是进行解析升级脚本的
* 而升级脚本也是在升级包中，也就是解析update-script进行升级的。
* 升级过程是基于文件升级的

```
|----boot.img
|----system/
|---META-INF/
    |CERT.RSA
    |CERT.SF
    |MANIFEST.MF
    |----com/
           |----google/
                   |----android/
                          |----update-binary
                          |----updater-script
                   |----android/
                          |----metadata

```

如上是升级包中基本的结构，大家可以自行解压升级包查看内部结构。下面着重说下`A/B分区升级`

### A/B(无缝)系统升级

主要就是每个分区存在两个副本(A和B)

引入目的就是降低升级后变砖的可能性，提高设备稳定性。

API支持：Android Nougat (API 24)

下面说下相关**优点**，便于理解谷歌这么做的目的：

* OTA 更新可以在系统运行期间进行，而不会打断用户。也就是在用户使用过程中，后台自动下载更新。不需要像`Recovery`模式那种需要重启就如升级页面，让用户等待升级过程。
* 更新后，重新启动所用的时间不会超过常规重新启动所用的时间。这就是说升级过程中已经把另一个分区进行擦写了，用户重启后直接进入升级后的分区，而无需等待解压覆盖等升级动作
* 如果 OTA 升级失败，用户当前使用的分区将不会受到影响。用户将继续运行旧的操作系统，并且后台也会继续尝试进行更新。
* 如果 OTA 更新已经成功但无法启动，设备将重新启动到旧分区，并且仍然可以使用。并且后台还会重新尝试进行更新。也就是说让用户保证使用一个可以正常启动的系统，而减少变砖导致用户无法使用的情况。
* 任何错误（例如 I/O 错误）都只会影响未使用的分区组，并且用户可以进行重试。
* 支持流式传输到 A/B 设备(边下边升)，因此在安装之前不需要先下载更新包。流式更新意味着用户没有必要在 /data 或 /cache 上留出足够的可用空间以存储更新包。这样对于用户控件比较少的设备是比较实用的。

个人觉得谷歌下这么大力气引入这个升级流程的目的，应该是对于目前市面上各个OEM和芯片厂家跟进谷歌大版本升级慢比较不满意，大家知道产品升级维护和售后是非常耗费时间和金钱的。作者做ATV项目，只要产品投放到市场，那么就要满足谷歌要求的至少3年进行版本迭代，不仅包括季度中的安全补丁更新，同时也要求至少2个大版本的更新维护。

谷歌更希望新老用户都能将手中的设备更新到最新版本，体验更多新功能，当然安全和体验也都上一个档次。

### 相关名词介绍

* 槽位slot (`slot A slot B`)，谷歌把两个分区分别称为`slot A` `slot B`,这也是A/B升级的名称由来
* `update_engine` 守护进程，是个可执行文件，位置在`system/bin/update_engine`
* A/B 分区中各个子分区命名方式一般都是如下方式命名（槽位的名称始终为 a、b 等）：

```
boot_a, boot_b
system_a, system_b
vendor_a, vendor_b
```


### SDK中相关配置

对于支持`AB升级`方式的SDK，相关配置与以前`Recovery`有所不同，如下进行简单介绍

谷歌对于配置有基本要求：详见文档[https://source.android.google.cn/devices/tech/ota/ab/ab_implement](https://source.android.google.cn/devices/tech/ota/ab/ab_implement)

以RTK平台为例`device/realtek/hank/device.mk`

![device.mk](/img/ab-update/device.mk.png)

如上图，是RTK的SDK配置情况支持`AB升级`，可以看到

* `AB_OTA_UPDATER := true` 必须支持的标志

* `AB_OTA_PARTITIONS := \` 生命支持的双分区名字

* `PRODUCT_PACKAGES += \` 其中`update_engine`就是实现升级逻辑的服务端了，这个必须要进行编译的

* `PRODUCT_PACKAGES_DEBUG += update_engine_client` 这个是进行测试服务端而进行调试的客户端，只在userdebug模式下才会进行编译

* `PRODUCT_PACKAGES += SystemUpdaterSample` 这个是专门给UI端进行测试用的apk，里面会讲解如何调用framework的api，以及处理相关返回值

  

### 分区相关

由于双分区，必然会额外占用闪存空间，谷歌也考虑到了这个问题[https://source.android.google.cn/devices/tech/ota/ab/ab_faqs#how-did-ab-affect-the-2016-pixel-partition-sizes](https://source.android.google.cn/devices/tech/ota/ab/ab_faqs#how-did-ab-affect-the-2016-pixel-partition-sizes)

![partition](/img/ab-update/partition.png)

上图中，谷歌以Pixel手机为举例，大致说明了一下与非AB系统相比，大约多了320MB的空间，作者在RTK平台(8GB Flash)，看到sdcard空间在AB系统与非AB系统分区上，差别基本不大(剩余都在4GB左右)，这与各个芯片厂家的配置有关。对于用户来说影响不大

下图中进行了分区表中相关分区的功能说明:

![partition-info](/img/ab-update/partition-info.png)

### 分区属性(状态)

在设备开机时，`bootloader`为了判断一个槽位(`slot`)是否为**可启动**的状态, 需要为其定义对应的**属性(状态)**

说明如下:

* **active** 活动分区标识, 排他, 代表该分区为启动分区, bootloader总会选择该分区
* **bootable** 表示该slot的分区存在一套可能可以启动的系统
* **successful** 表示该slot的系统能正常启动
* **unbootable** 代表该分区损坏, 无法启动, 在升级过程总被标记, 该标记等效于以上标记被清空. 而active标记会将该标记清空

`slot a` 和 `slot b`, 只能有一个是`active`, 但它们可以同时有 `bootable` 和 `successful` 属性

网上找了一个图片，很形象的说明了从B分区升级后进入A分区的过程中各个属性是如何使用的

![property](/img/ab-update/property.png)

### Sample apk 解析

源码路径：`bootable/recovery/updater_sample/`

大致流程如下：

1. 创建UpdateEngine实例
2. 绑定监听回调
3. 执行applyPayload方法

状态机流程如下：

![ui-state-machine](/img/ab-update/ui-state-machine.png)

//TODO 介绍源码



### Bootloader & Fastboot

目前Android 高版本中`fastboot`拆分成两种模式`bootloader` 和`fastbootd`

**Bootloader模式**

进bootloader方法：`$ adb reboot bootloader`(或者串口中`reboot bootloader`)

Bootloader模式下一般支持如下指令(win的cmd窗口执行如下命令)

```
fastboot reboot -->设备重启
fastboot reboot-bootloader -->设备重启进bootloader
fastboot reboot-fastboot -->设备重启进fastboot
fastboot flashing unlock_critical -->设备解锁
fastboot flashing unlock   -->设备解锁
```

**Fastbootd 模式**

功能主要是刷各个系统分区的

进fastboot方法：`adb reboot fastboot`（或者串口`reboot fastboot`）

支持部分命令如下：

```
fastboot reboot -->重启进主系统
fastboot reboot-bootloader -->重启进bootloader
fastboot getvar all  -->支持的命令与参数列表
fastboot flash system system.img  -->刷system.img分区
```

### 动态切换slot

出厂时默认·slot a· 和·slot b·都是完整的，开发中如何手动切换slot呢，下面介绍方法

步骤1：查看当前`slot`

	方法1：
	console:/ # getprop |grep slot 
	[ro.boot.slot_suffix]:[_a] -->可以看到当前槽位是a
	
	方法2：或者进入fastboot 模式后 
	C:\Users\nathan> fastboot getvar current-slot 
	current-slot: a -->可以看到当前槽位是a

步骤2:解锁

```
进入bootloader进行解锁
(1)C:\Users\nathan> adb reboot bootloader -->进入bootloader
(2)C:\Users\nathan> fastboot flashing unlock -->执行解锁指令
```

步骤3：切换

```
进入fastboot模式
C:\Users\nathan> fastboot reboot-fastboot -->进入fastboot模式
或者
C:\Users\nathan> adb reboot-fasboot
指令：  
C:\Users\nathan> fastboot set_active a -->使a槽位变成active
或   
C:\Users\nathan> fastboot set_active b -->使b槽位变成active
```



### Code & Doc

主要代码和文档汇总

谷歌在线文档：[https://source.android.google.cn/devices/tech/ota](https://source.android.google.cn/devices/tech/ota)

Sample app路径：`bootable/recovery/updater_sample`

源码路径 ：`/system/update_engine`

Framework API路径：

`frameworks/base/core/java/android/os/UpdateEngine.java`

`/frameworks/base/core/java/android/os/UpdateEngineCallback.java`



全文 完！









