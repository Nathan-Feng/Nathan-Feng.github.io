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

### 解决问题

本文主要解决的问题有：

1. system app可以调用`system/bin`和`/vendor/bin`中的可执行程序，并能拿到返回结果
2. system app和HIDL服务可以共享某个目录的文件读写权限(打通framework和HAL之间的壁垒，不过谷歌并不希望这么做)

### 前置条件

本文主要涉及如下知识内容(有了解就好，不要求深入)，如果不了解的，可以自己先补一下：

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

由于谷歌在Android 8.0上引入了Treble架构,进一步隔离了 Framework进程和Hal，所以system分区里面的app想要调用vendor里面的接口，必须走HIDL，本文也是从HIDL入手，利用HIDL的service权限，来提升App调用的权限。

### 调用流程

首先总体流程如下:

![overall-flow](/img/shell-cmd/overall-flow.png)

- App用于发送执行shell命令，并接受和处理返回结果
- API：主要是一个封装好的jar包，动态或者静态均可
- JNI层：把shell命令和结果传递给Native层
- HIDL Client ：主要是获取HIDL接口服务，并进行调用
- HIDL层：封装接口便于system和vendor层hal之间通信
- HIDL Server：实现HIDL层接口
- API impl：真正执行和处理shell 命令的地方

图中右边我也标出来各自编译后的生成物位置，便于大家宏观理解，后续会把生成物具体内容贴出来。

下面会分几部分讲解代码示例，包括接口设计，HIDL服务创建，C层具体实现，最后是SELinux权限处理等。

关于代码位置我会再单独出一篇文章，讲讲我在ROM开发过程中一些常用方法介绍。

### 接口设计

源码入口：[https://github.com/Nathan-Feng/shell-hidl](https://github.com/Nathan-Feng/shell-hidl)

#### 1.Jar

````
ShellCmd.java
	public native String hsInvokeJni(String cmd, int type);
````

这里参数`cmd` 就是需要执行的shell命令，`type` 主要是用于区分是否需要返回值，因为有些shell命令是阻塞式的，如果不区分的话，很容易卡住，返回值就是shell执行的结果。

另外由于是要调用JNI方法，所以这里加了`native`。

#### 2. Shell命令的C实现

```C
nathan_hal.c

static int do_system_cmd(const char *arg, char *reply,bool isNeedRet){
	FILE *fpRead = NULL;
	char buf[1024]={0};
	memset(buf,0,sizeof(buf));
		fpRead = popen(arg,"r");
		if(fpRead == NULL){
			ALOGE(" run system error!");
			return -1;	
		}
		if(!isNeedRet){
			pclose(fpRead);
			fpRead = NULL;
			return 0;
		}
		while(fgets(buf,1024-1,fpRead) != NULL && strlen(reply) < BUFFER_MAX){
			if(BUFFER_MAX > (strlen(buf) + strlen(reply))){
				strncat(reply,buf,strlen(buf));
			}				
			else{			
				strncat(reply,buf,BUFFER_MAX - strlen(reply));
			}
		}	
		pclose(fpRead);
		fpRead = NULL;
	return 0;
}
```

主要就是利用popen函数创建管道来发送shell命令，并拿到fget拿到返回结果。

其他主要看代码就好，这里就细说了。

#### 3.HIDL服务创建

关于HIDL服务，我这里就说一点权限问题

```c
vendor.nathan.hwstbcmdservice@1.0-service.rc

service hwstbcmdservice-1-0 /vendor/bin/hw/vendor.nathan.hwstbcmdservice@1.0-service
    class hal
    user root
    group root system

```

其中user和group，我加的root和system，如果user是system的话，那么这个服务是无法执行root shell命令的。这里要注意下。

### 4.SELinux(SEAndroid)权限处理

谷歌官网地址：[https://source.android.google.cn/security/selinux](https://source.android.google.cn/security/selinux)

我这里主要涉及的`*.te` 如下：

```
-- sepolicy
   |-- file.te  -->主要是声明一个可读写的目录，用于system_app读写文件，以及HIDL层的服务读写文件来使用
   |-- file_contexts   -->1.声明/data/vendor/stbdata目录的标签上下文属性，2.声明HIDL服务的上下文标签属性
   |-- hal_hwstbcmdservice.te  -->HIDL服务的domain定义域与权限处理的主要内容
   |-- hwservice.te   -->声明hal_hwstbcmdservice这个服务是注册到hwservice_manager里面的
   |-- hwservice_contexts  -->声明HIDL接口服务的上下文标签的
   |-- system_app.te   -->处理system_app的调用权限的
```

如上这些熟悉HIDL服务创建的童鞋会比较了解

下面说下`hal_hwstbcmdservice.te`这个权限添加过程：

1. 我一般先临时关闭掉selinux，`setenforce 0` 然后调用接口
2. 抓avc的打印：`dmesg |grep avc` 可以查找与调用命令相关的权限，加入到.te中
3. 添加.te完毕后，整编Andorid，如果出错，那就说明违反了谷歌政策，需要找找其他解决方案，比如是否是权限过大，例如`ioctl`,此时需要`allowperm`加上具体要添加的权限才行，具体大家可以自行百度
4. 然后在`out/target/product/XXX/vendor/etc/selinux/vendor_sepolicy.cil`里面找下是否存在，然后烧录img来验证
5. 谷歌也提倡最小权限原则，也就是只要能实现目的，就别多加额外权限，保障系统安全，人人有责

#### 5.共享文件夹问题

有些时候，执行的shell命令需要输出结果到某个文件中供system app读取，而有些时候，system app需要保存一些网络数据等到某个文件夹中，以便shell命令能读取和执行,不过sdcard等目录可能安全性不高，容易别发现。此时，就需要我们自定义一个目录(详见`file.te`)

以`tcpdump`命令为例：`"/vendor/bin/tcpdump -i any -s 0 -w /data/vendor/nathandata/1.pcap`

命令中-w参数需要指定一个目录，当tcpdump把结果输出到目录后，system app需要读取，或者分析，或者上传。那么由于这个文件是root shell创建的。自然1.pcap的权限是`root:root` 

解决方法：通过jar，调用`chown` 修改文件所有权为`system:system`即可，

反之如果是system app保存的文件，那么想让root shell读取的话，那么就改成`root:root`呗

以上，亲测都是OK的。



#### 总结

	- 没有万能的接口，都是遇到具体需求，具体分析的
	- 本文的例子是本人`智慧`的结晶，如果满意，还请三连
	- 源码传送门：[https://github.com/Nathan-Feng/shell-hidl](https://github.com/Nathan-Feng/shell-hidl)

下一篇介绍下相关编译的配置部分，欢迎大家点击阅读





