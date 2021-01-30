---
layout:     post
title:      "Android ROM 开发之SDK常用开发适配方法总结 --Base on Android 10.0"
subtitle:   "快速跨平台移植，减少适配工作量"
date:   2021-01-24 14:25:00 +0800
author:     "Nathan"
tags:

    - ROM
    - SDK

---

> 步伐不必太大，只要方向正确就好
>
> 既然想把世间精彩尽收眼底，就必须踏出舒适圈
>
> Life's so short and such glorious images in the world, and such horror as whell, but I want to see it all. So I've moved out of my confort zone.



### 前言

做了ROM开发也有一段时间了，平台SDK方面开发维护方面也多少积累了一些经验，现在分享给大家，希望对大家有帮助

### 问题原因

针对Andorid TV产品开发，SDK的生命周期一般比较短，一年一个大版本，所以不仅要应对每年的新版本带来的适配问题，同时也要应对不同的芯片平台带来的SDK差异。这样无形会增加平台移植的工作量，所以我就研究了一下SDK开发适配方面的内容，以减少适配时间为目标。本文的目的也是给大家一个思路，完全是个人总结，后续也会继续完善。

### 解决问题

本文主要解决的问题有：

1. 如何预制apk，jar，so，等所有文件并编译打包到ROM里
3. 如何自定义自己的厂家目录，跨芯片移植时省时省力
4. 开发问题定位

#### 1.预置文件到ROM

无论是添加Android.mk,Android.bp等声明了需要编译的文件后，都需要在`/device/manufacturer/device-name/device.mk` 里面加入`PRODUCT_PACKAGES += xxx` ,这样当make的时候才会真正打包到ROM里

预置apk(有源码)

```
LOCAL_PATH:= $(call my-dir)  //获取当前目录
include $(CLEAR_VARS) //清除一些LOCAL_XXX变量，用于此次编译，
LOCAL_MODULE_TAGS := optional //user debug eng tests optional ，指定编译在哪个版本下，optional是任何版本都编译
LOCAL_SRC_FILES := $(call all-subdir-java-files) //编译时加载当前目录下的所有java文件
LOCAL_PACKAGE_NAME := Test //最终生成system/app/Test/Test.apk
LOCAL_JAVA_LIBRARIES := shard-jar //引用动态jar包，比如system/framework里面的jar
LOCAL_STATIC_JAVA_LIBRARIES := static-jar  //引用静态的jar，此时jar会打包到apk中
LOCAL_PRIVATE_PLATFORM_APIS := true //会使用sdk的hide的api来编译
LOCAL_SDK_VERSION := current  //意思是编译时忽略系统隐藏类(@hide) ,不同与LOCAL_PRIVATE_PLATFORM_APIS同时用
LOCAL_PROPRIETARY_MODULE := true //控制生成到system还是vendor分区里
LOCAL_MODULE_RELATIVE_PATH := hw //控制相对路径，一般用在编译so时使用
LOCAL_PRIVILEGED_MODULE := true，//控制app放在/system/priv-app中
include $(BUILD_PACKAGE) //编译apk
```

预置apk(无源码)

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := Test //生成/system/app/Test/Test.apk
LOCAL_MODULE_TAGS := optional 
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk //需要预置的源文件名字
LOCAL_MODULE_CLASS := APPS //选项有APPS，JAVA_LIBRAYIES，SHARED_LIBRAYIES，EXECUTABLES，看名字就知道啥意思了吧
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX) 
LOCAL_CERTIFICATE := PRESIGNED //编译时不签名，如果是platform，就是平台签名
LOCAL_MULTILIB :=32 //在32位平台运行
LOCAL_OVERRIDES_PACKAGES := MyTest //编译时排除Mytest.apk
include $(BUILD_PREBUILT) //预编译
```

预置so(有源码)

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := test-jni // 生成system/lib/test-jni.so
LOCAL_SRC_FILES := test-jni.c //源文件
LOCAL_SHARED_LIBRARIES := shared-lib //引用system/lib/shared-lib.so
include $(BUILD_SHARED_LIBRARY)
```

预置so(无源码)

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := Test //生成/system/app/Test/Test.apk
LOCAL_MODULE_CLASS := SHARED_LIBRARIEDS // 
LOCAL_SRC_FILES := $(LOCAL_MODULE)/test.so //需要预置的源文件名字
LOCAL_MODULE_SUFFIX := .so 
LOCAL_MULTILIB :=32 //在32位平台运行
include $(BUILD_PREBUILT) //预编译
```

#### 2.Android 源码中常用的编译规则说明

我们知道Android编译步骤就三个`source build/envsetup.sh` , `lunch product` , `make -j16` 

然后就进入了`/device/manufacturer/device-name/device.mk` 中，本文就从它开始吧。

##### **PRODUCT_PROPERTY_OVERRIDES** 

用于在vendor/build.prop里面生成一些属性，也就是键值对

```
PRODUCT_PROPERTY_OVERRIDES += ro.aa.bb=cc   //会在vendor/build.prop 生成ro.aa.bb=cc
```

##### **PRODUCT_PACKAGES**  

定义需要编译打包的文件名字,比如apk.so,xml等

```
PRODUCT_PACKAGES += Test //查询所有Android.mk或者.bp中LOCAL_MODULE(LOCAL_PACKAGE_NAME)声明为Test,并编译打包
```

##### **$(call inherit-product-if-exists, vendor/xxx/yyy/zzz.mk)** 

用于包含zzz.mk，就像cd到某个目录里去编译一样

##### **`BoardConfig.mk`中常用的规则用法**

`include /vendor/aaa/bbb/ccc.mk` //用于包含ccc.mk，

`BOARD_SEPOLICY_DIRS` 用于声明.te需要包含的目录

```
`BOARD_SEPOLICY_DIRS += vendor/mm/nn/sepolicy   ///包含另一个有.te策略的目录
```

`DEVICE_MANIFEST_FILE`,`DEVICE_MATRIX_FILE`,`DEVICE_FRAMEWORK_COMPATIBILITY_MATRIX_FILE`这些主要是声明HIDL注册xml的

### 定义通用目录，提高跨平台移植

一般我们拿到芯片厂家提供的SDK源码后，通常做法都是在芯片厂家的**源文件**基础上，进行**增删改**之类的，

那么带来的问题，就是换个SDK，或者换个平台，那么补丁都需要重新对比合入，比较费时费力，还容易出错

下面介绍我的方法，大家可以参考：

1. 由于供应商目录/vendor/nathan
2. 定义两个mk: /vendor/nathan/nathandevice.mk, 和vendor/nathan/nathanboradconfig.mk
3. 在芯片厂家的源码目录的`/device/manufacturer/device-name/device.mk`中最后一行加入`$(call inherit-product-if-exists,  /vendor/nathan/nathandevice.mk)` 
4. 在芯片厂家的源码目录的`/device/manufacturer/device-name/BoradConfig.mk`中最后一行加入`include /vendor/nathan/nathanboradconfig.mk)` 
5. 通过这种方式，移植任何SDK，只需要修改两行即可，
6. 在自定义的device.mk中都可以加上常用的属性配置等。大大方便开发和调试

### 开发问题定位

找个根据平台不同，调试定位方法也差异，不过android部分很多通用的命令和方法都是通用的，

这些在开发中都可以逐渐积累，

我的观点就是：**遇到问题，分析问题，解决问题，总结问题，发散问题，优化问题**















