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

我们平时登陆 Linux 系统时，虽然输入的是自己的用户名和密码，但其实 Linux 并不认识你的用户名称，它只认识用户名对应的 ID 号（也就是一串数字）。

Linux 系统中，每个用户的 ID 细分为 2 种，分别是用户 ID（User ID，简称 `UID`）和当前工作组 ID（Group ID，简称 `GID`），这与文件有拥有者和拥有群组两种属性相对应，另外还有一个用户所有组ID(Groups ID，简称`GIDS`)

Linux `id`命令用于显示用户的ID，以及所属群组的ID

```
pi@raspberrypi:~ $ id
uid=1000(pi) gid=1000(pi) groups=1000(pi),4(adm),20(dialout),24(cdrom),27(sudo),29(audio),44(video),46(plugdev),60(games),100(users),105(input),109(netdev),997(gpio),998(i2c),999(spi)
```

如上，可以看到我的树莓派当前`uid`是1000，当前工作组`gid`是1000，当前用户所属的所有组的`gids`是后面那一长串

更多`id` 命令用法，请用`id --help`

同样，linux**进程**也有`uid,gid,gids`，某个用户启动的进程，那么这个进程就会继承用户的`uid,gid,gids`

查看命令为`ps -aux`

```
pi@raspberrypi:~ $ ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         2  0.0  0.0      0     0 ?        S    09:43   0:00 [kthreadd]
root       510  0.2  0.4  32796 17400 ?        S    09:43   0:05 /usr/bin/vncserver-x11-core -service
pi         617  0.0  0.0   9932  3096 ?        Ss   09:43   0:00 /usr/bin/vncserver -depth 16 -geometry 1024x768 :1
pi         782  0.0  0.0   9788  2556 pts/0    R+   10:24   0:00 ps -aux 
pi        1638  0.0  0.0   8516  3776 pts/0    Ss   09:43   0:00 bash
```

如上截取部分输出，可以看到我用当前用户`pi`执行`ps -aux`这个执行，那么其进程的`uid`（USER那一列为`pi`）

最后介绍linux文件系统，只有`uid gid` 以及相对应的`rwx`权限

查看命令为`ls -l` 可以看到文件型态、权限、拥有者、文件大小以及时间等内容

```
pi@raspberrypi:~/Downloads $ touch abc
pi@raspberrypi:~/Downloads $ ls -l
total 0
-rw-r--r-- 1 pi pi 0 Jul 18 10:32 abc
pi@raspberrypi:~/Downloads $ sudo touch bbb
pi@raspberrypi:~/Downloads $ ls -l
total 0
-rw-r--r-- 1 pi   pi   0 Jul 18 10:32 abc
-rw-r--r-- 1 root root 0 Jul 18 10:32 bbb
pi@raspberrypi:~/Downloads $ 
```

如上，我在当前目录下使用当前用户`pi`创建了一个文件，那么从结果来看`-rw-r--r-- 1 pi pi 0 Jul 18 10:32 abc`我们就知道了这个文件的`uid`为pi ,`gid`为pi，其`rwx`权限为`-rw-r--r--`,其中第一个横杠`-`代表是一个文件,其余的9位，以3位为一组，分别代表当前用户对着文件的所有权，当前用户组对这个文件的所有权，以及其他用户组对这个文件的所有权，

当我切换到root用户去创建一个文件夹的时候，那么可以看到其`uid gid`也变成了root

总结一下：

* 系统上的每个进程(运行中的程序)都作为一个特定的用户来运行
* 每个文件都归一个特定的用户所有
* 对资源(文件和目录等)的访问受用户所限制
* 正在运行的进程所关联的用户可以决定该进程可访问的资源(文件/目录等）

### DAC介绍

全称：Discretionary access control (DAC) 自主式权限控制

概念：如果一个用户拥有一个文件，那么用户可以允许自由的控制文件的读写和执行权限，那么就叫DAC

上一节介绍的Linux标准安全机制就是属于DAC，那么这个机制有什么特点(缺点)吗

1. 资源所有权掌握在特定用户的手里，导致资源可能被乱用
2. 用户的区分度比较低，系统上只有两个特权等级：普通user和超级用户root
3. 读权限可以转移，导致信息可以任意流动，比如用户A允许文件被B用户读，那么B读到文件后，可以转手传递给C

可以看到这种DAC模式无论是用户和资源都难以控制，因此大家都想疯狂拿到root权限，然后就可以为所欲为

### Android沙箱介绍

Application Sandbox，官网地址：[应用沙盒](https://source.android.google.cn/security/app-sandbox)

简单来说主要是

* 基于Linux保护应用资源

* 每个app分配唯一的UID/GID,在各自的的进程中运行

* 互相不能访问(默认)

既然Android是基于Linux的，那么进程和资源也有`uid/gid/gids`,下面就介绍一下

* 每个app安装之后都会被分配一个`uid`叫`aid`，源码路径`/system/core/include/private/android_filesystem_config.h`

* `id`查看uid/gid，命令没变,`cat /proc/xxx/status`查看进程信息，包括uid、gid、gids

* `ps -l`查看进程`uid/gid` 比如 `u0_a36`那么意思就是10000 add 36 为10036, 其中`#define AID_APP          10000  /* first app user */`

* 资源或者说文件的`uid/gid`定义路径:`/system/core/include/private/android_filesystem_config.h`

* 最后说下Android 服务service，其一般会伴随着一个xxx.rc文件,并在其中声明其uid/gid,例如下面中声明user和group，对应的就是uid和gid

  ```
  service surfaceflinger /system/bin/surfaceflinger
      class core
      user system
      group graphics drmrpc readproc
      onrestart restart zygote
      writepid /sys/fs/cgroup/stune/foreground/tasks
  ```

* 另外设备节点如`/dev/binder `的uid和gid一般定义在各个ueventd.rc文件中 一般放在`/system/etc/ueventd.rc`下

那么Android中的权限Permission除了上面介绍的rwx，对于APK开发，还包括清单文件中需要配置的权限，路径：`/frameworks/base/core/res/AndroidManifest.xml`

无论是通过Android API，Java API，NDK C API，还是执行shell命令，那么要么在api调用过程中，如framework，要么是在底层，通过uid/gid等进行权限管理和控制。道理是相通的。


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
