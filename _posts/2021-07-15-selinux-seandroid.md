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

全称Security-Enhanced Linux (SELinux) ，安全增强型Linux，是增强安全的一个机制，那么就探究下到底相比DAC模式，如何增强的？

首先说下背景知识，简单了解下：

* 是集成在kernel 2.6.x中一个安全架构
* 是Linux Security Modules (LSM)的一部分
* 是美国国家安全局和linux社区开发的一个项目
* 是MAC (Mandatory Access Control)在Linux内核级的一个实现

其实最重要的就是它是MAC的一个linux实现，那么MAC是什么，跟DAC什么关系呢？

### MAC介绍

* MAC被预置到内核级的系统中，对于linux，是预置在kernel里面的
* MAC限制**主体**(英文叫subject，比如用户或进程)对**客体**(英文名叫object，比如文件等资源)的**访问**和**操作**的能力
* MAC机制会给**所有**主体和客体分配一个**安全标签**(Security label)，包括用户user，进程process，访问的资源等
* MAC机制会定义一个清晰明确的的访问**规则**，限制每个用户或者进程只允许访问它**已经定义好**的资源，所以即使是Root权限也能被限制
* 是非DAC的，不能被资源的所有者修改

如上可以看出，我们想要了解SELinux以及相关的概念，那么会出现很多名词，当然这些名词其实只是在不同场景下的不同叫法而已，大家习惯就好，我也是经常用，所以有时候会说出好几个名词，但是其实都是表达一个意思。

### SELinux架构流程

![selinux-arc](/img/selinux-seandroid/selinux-arc.png)

我这边画了一个图，大致说下流程，首先主体(Subject比如user或Process)想要访问(Access) 一个客体(Object，比如一个文件file吧),那么首先需要经过DAC判断，如果DAC判断失败了(比如rwx权限不足等)，那么直接就会拒绝

如果DAC验证通过，那么就会进入MAC的判断，其中MAC，我又把具体内容扩展了一下，见图中虚线方框内，

进入MAC时，首先会通过AVC(访问向量缓存)，看名字cache缓存，那么缓存什么呢？缓存的是Security Server(相当于数据库) 对于主体访问客体的策略是否命中，如果命中了，那么为了提高判断效率，直接会把这个策略放到AVC里缓存，下次判断时，直接缓存里查找，速度更快。

最后通过AVC检查后，才能得到访问客体的权限，拿到相应的资源。我这里省略了图中的MLS，这个在后续会单独介绍。

如上可以知道，DAC和MAC是互为补充的，先要经过DAC后才能进行MAC，所以SELinux也叫增强安全型

### SEAndroid介绍

了解过SELinux之后，那么SEAndroid就比较好理解了，其实：

SELinux    +   Android    =    SEAndroid

也就是说安卓平台对于SELinux的一个实现，或者应用于安卓上，就叫SEAndroid

官网地址：[https://source.android.google.cn/security/selinux](https://source.android.google.cn/security/selinux)

简单说下历史演变过程：

* Android 4.3（宽容模式）
* Android 4.4（部分强制模式）
* Android 5.0 及更高版本中，已全面强制执行 SELinux
* Android 6.0 高度限制对 /proc 的访问
* Android 8.0 更新了 SELinux 以便与 Treble 配合使用

这里解释下宽容模式（Permissive）和强制模式（Enforcing），SELinux 按照默认**拒绝**的原则运行：任何未经明确允许的行为都会被拒绝。SELinux 可按**两种**全局模式运行：

* 宽容模式：权限拒绝事件会被记录下来，但不会被强制执行。
* 强制模式：权限拒绝事件会被记录下来**并**强制执行。

### Android CDD

提到安卓，不得不提CDD兼容性文档，谷歌认证必备的，

CDD中对于SELinux部分也是有强制要求的，具体链接如下：[https://source.android.google.cn/compatibility/11/android-11-cdd#9_7_security_features](https://source.android.google.cn/compatibility/11/android-11-cdd#9_7_security_features)

这里我直接粘贴过来翻译过的内容

```
如果设备要使用 Linux 内核，则：
•[C-1-1] 必须实现 SELinux。
•[C-1-2] 必须将 SELinux 设置为全局强制模式。
•[C-1-3] 必须将所有域配置为强制模式。不允许使用宽容模式域，包括特定于设备/供应商的域。
•[C-1-4] 对于 AOSP SELinux 域以及特定于设备/供应商的域，不得修改、省略或替换上游 Android 开源项目 (AOSP) 中提供的 system/sepolicy 文件夹中存在的 neverallow 规则，并且政策必须在所有 neverallow 规则都存在的情况下编译。
•[C-1-5] 必须在每个应用的 SELinux 沙盒中运行面向 API 28 级或更高级别的第三方应用，并对每个应用的私有数据目录设定应用级 SELinux 限制。
•应保留上游 Android 开源项目的 system/sepolicy 文件夹中提供的默认 SELinux 政策，并且应仅针对自己的设备特定配置向该政策进一步添加内容。
```

可以看出，谷歌对于安卓系统的要求还是挺严格的，当然国内厂家如果不需要满足CDD的话， 很多都是为了拿到更多权限而直接宽容模式运行的，但是这样的话，系统安全就得不到保证，所以说安全和权限总是鱼和熊掌不可兼得

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
