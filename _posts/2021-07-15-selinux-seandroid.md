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

### 自编故事

很久以前，有个公司成立了，公司里有总经理张总一人，秘书小王一人，还有李赵等小兵若干，有一天总经理写了一封信，信中写了公司员工每个人的薪水等级，写完后，放在一个柜子里，

结果总经理为了方便，设置了柜子权限为老铁666，然后下班回家睡觉了

秘书小王有事进到张工办公司发现没人，本来想离开的，但是看到屋里有个柜子，上面钥匙挂在上面(可读写)，小王就忍不住打开偷看了一下，看完之后，心生一计，偷偷改了自己的薪水等级，然后关上柜子走了

小李小赵有事请教来找张总，结果也发现了柜子，然后也偷偷改了数据(可读写)，

等第二天张总上班后，再拿出来信的时候，倒吸一口凉气，大事不妙。。。。

心想，这种不小心就把文件的权限暴露给其他人的情况，太不安全了，必须要改革！

张总熬了几个通宵，掉了一把头发，终于新的机制2.0诞生了,

2.0模式是这样的，公司里面的每个人都有一个公司派发的身份证，每个文件，也有一个身份证，公司还雇用了一个保安大爷(董事长化身)，实时监控每个人对于每个文件的行为。

首先张总写信的时候，还是保持以前的模式，每次都检查下其他人是否有可读写的权限，

然后小李在打开柜子想要读写信的时候，保安是会立即检查小李的身份证和信的身份证，然后在一台只能读的电脑上，搜索小李的权限，看看人有没有读写信的权限

如果没有的话，那么保安会立即传送到小李身边，揪住小李拖出去，然后利用公司大喇叭广播所有人，小李想读信，被我阻止了，大家下次别干了啊，

张总每天最开心的事，就是听保安喋喋不休的说谁谁谁没干成什么什么事。。。。

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

SELinux 引入**标签(label)**这个东西来进行权限的操作和规则的制定，在SELinux世界里，任何一个对象都有标签，比如进程，文件，目录等等，全都有标签，就像每个人身份证上的名字一样，与生俱来，而且每个对象的标签都是唯一的。SELinux 在做决定时，就会根据这些标签以及系统根据标签来制定的规则进行权检查。

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

### Android  CDD

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

### SEAndroid 标签Label

![selinux-label](/img/selinux-seandroid/selinux-label.png)

下面介绍重要概念**标签label**，我这里又画了一个图，便于理解这个概念，首先label是给所有对象贴的，包括进程和资源

首先标签label又叫`Security Context`安全上下文，然后进程的标签，我们也叫`scontext(source context)源上下文`，而资源的标签也叫`tontext(target context)目标上下文`名字如意思，比较好理解。

然后它长什么样子呢，就是`user:role:domain或者type:mls_level` 也就是三个**冒号**分隔成四个部分

其中`user`叫用户，在SEAndroid中只定义了一个用户叫`u`,

然后`role`角色，在SEAndroid中也定义了两个角色叫`r和object_r`

什么区别的，`r`是给进程用的，而`object_r`是给访问的对象或者叫资源用的，因为object就是对象的意思

然后`domain或者type`什么区别呢？如果进程的标签，那么就是`domain域`,如果是资源，那就是`type`

最后是`mls_level`在SEAndroid里面只定义了一个级别`s0`

其实从标签的样子,大家可以知道，四部分中主要变化的是第三部分(domain/type)，所以安卓中主要以这个部分定义相关规则

SEAndroid 的标签大致分为五类：

* 首先是Service服务相关的标签。
* 其次，对于基于 Binder 的服务，允许向 Service Manager 注册的标签。
* 第三，系统属性Property的标签。
* 第四，设备节点相关的标签。
* 第五，文件相关的标签
* 最后，app应用相关的标签。

这里先说下SEAndroid中文件相关的标签

* file_contexts  为文件分配标签，为/system /sys /dev /data 等文件分配标签
* genfs_contexts  用于为不支持扩展属性的文件系统（例如，proc 或 vfat）分配标签
* property_contexts 用于为 Android 系统属性分配标签。在启动期间，init 进程会读取此配置。
* service_contexts 用于为 Android Binder 服务分配标签，以便控制哪些进程可以为相应服务添加（注册）和查找（查询）Binder 引用。在启动期间，servicemanager 进程会读取此配置。
* seapp_contexts 用于为应用进程和 /data/data 目录分配标签。在每次应用启动时，zygote 进程都会读取此配置；在启动期间，installd 会读取此配置。
* mac_permissions.xml 用于根据应用签名和应用软件包名称（后者可选）为应用分配 seinfo 标记。随后，分配的 seinfo 标记可在 seapp_contexts 文件中用作密钥，以便为带有该 seinfo 标记的所有应用分配特定标签。在启动期间，system_server 会读取此配置。

最后说下如下的信息含义

```
W com.aa.bb: type=1400 audit(0.0:199): avc: denied { call } for  scontext=u:r:system_app:s0 tcontext=u:object_r:update_engine:s0 tclass=binder permissive=0 app=com.aa.bb
```

我们平时一般会通过logcat查看相关打印信息，这里介绍两个命令来抓取selinux的拒绝事件打印：`logcat |grep avc 或者dmess |grep avc` 就会过滤出一大堆类似如上的信息，我们要从这些信息中提取出有用的关键信息，并修改系统的规则来满足我们产品的要求。

下面我会介绍SELinux的规则，并解释如上的信息含义

### SEAndroid  规则1 Rule/Policy

每个对象都有标签了，那么如何利用这些标签来干事情呢，重要的**规则或者叫策略**登场了

基于安卓源码，介绍下SDK中源码相关的内容：

`system/sepolicy` 是android 关于selinux核心策略配置的所有内容，我们不应该去修改这个路径下任何文件

`/device/manufacturer/device-name/sepolicy` 这个路径会包含芯片厂家所有平台相关的配置，我们主要修改这里
 `BOARD_VENDOR_SEPOLICY_DIRS += vendor/oem/sepolicy` 我一般会把自定义的规则放在自定义的目录里面，便于跟芯片厂家的分区开

`*.te`策略配置文件的后缀，其中的语言从`global_macros, te_macros attributes` 这些配置文件中读取和使用

Android 8.0 之后

* system/sepolicy/public  平台共有策略的全部定义
* system/sepolicy/private 平台私有规则，不会向vendor部分暴露。
* system/sepolicy/vendor 厂商规则，可引用public的规则，不能引用private的规则
* device/manufacturer/device-name/sepolicy 厂商自定义的规则，包括如上的vendor部分

总之大家没事可以多去上面说的几个目录下点击文件看看内容，其中厂家一般都只修改device里面定义的规则，否则可能会引起谷歌认证失败的情况。大家还是遵守规则比较好

### SEAndroid 规则2

![image-20210722214227470](/img/selinux-seandroid/selinux-rule.png)

下面介绍下`*.te`中常用的语法规则，目的是让大家能看到，会写

上面图中以`adbd.te`为例，介绍具体的内容

首先说下domain域定义的规则，如下面两行

```
type adbd, domain; //解释下type就是定义的意思，定义adbd 为domain，看到domain，意思就是adbd是给进程或者用户贴的标签类型
type adbd_exec, exec_type, file_type; //这行意思是定义一个adbd_exec这个类型，然后属于exec_type和file_type,意思就是adbd这个文件的标签为adbd_exec，那么它是可执行文件(exec_type)，也是文件类型(file_type)
//图中标注为红色的，都可以通过箭头找到原始定义的源码路径，大家可以自己查看
```

域定义完了之后，紧接着就是定义规则了

```
allow domains types:classes permissions; //这行是语法规则，意思就是允许(allow) 某个域(domains) 对于某个资源(types)：资源类型(classes) 拥有资源类型相关的权限(permissions)
```

举个例子

```
allow bootanim system_file:dir r_dir_perms;
允许  域类型为bootanim的进程 对于资源为system_file的目录dir 有读r权限
```

另外介绍下有4种情况：

* allow - 允许主体对客体执行许可的操作。
* neverallow - 表示不允许主体对客体执行制定的操作。
* auditallow - 表示允许操作并记录访问决策信息。
* dontaudit - 表示不记录违反规则的决策信息，切违反规则不影响运行。

### SEAndroid 规格3

说下SDK编译这些规则最终的产出物策略生成位置

* `boot.img`（针对非 A/B 设备）或 `system.img/vendor.img`（针对 A/B 设备）。
* (/system|/vendor|/product)/etc/selinux  所有分区中的策略配置文件位置,包括如下内容
* (plat|vendor|product)_sepolicy.cil: 所有策略转成cil (CIL(common Intermediate Language))
* *_file_contexts: 文件的security contexts  其中`*` 代表`plat|vendor|product`
* *_property_contexts: 属性的security contexts
* *_seapp_contexts: App 的security contexts
* *_mac_permissions.xml 

### SEAndroid 命令

平时开发中，我这里总结了常用的命令来查看或者设置SELinux相关的内容

| **Command**                            | **Action**                                                   |
| -------------------------------------- | ------------------------------------------------------------ |
| setenforce 0/1 或 getenforce           | 临时关闭或打开安全策略 ，或者获取当前策略 Permissive/Enforcing |
| Ls -Z                                  | 显示文件的Security  Context                                  |
| ps -Z                                  | 显示进程的Security  Context                                  |
| id (user)                              | 显示进程/用户的Security Context                              |
| chcon                                  | 修改文件的Security  Context                                  |
| restorecon                             | 恢复文件的默认Security  Context                              |
| dmesg \|grep avc 或  logcat \|grep avc | 查看所有denied的信息                                         |

如下是一些实际操作的效果

```
console:/sdcard # ps -AZ
LABEL                          USER            PID   PPID     VSZ    RSS WCHAN            ADDR S NAME                       
u:r:init:s0                    root              1      0   46160   6772 SyS_epoll_wait      0 S init
u:r:kernel:s0                  root              2      0       0      0 kthreadd            0 S [kthreadd]
u:r:netd:s0                    root            306    281   17776   2632 pipe_wait           0 S iptables-restore
u:r:audioserver:s0             audioserver     332      1   58772  17608 binder_ioctl_write_read 0 S audioserver
u:r:su:s0                      root           5414   5149   10864   2932 0                   0 R ps
```

```
console:/sdcard # id system
uid=1000(system) gid=1000(system) groups=1000(system) context=u:r:su:s0
console:/ $ id 
uid=2000(shell) gid=2000(shell) groups=2000(shell),1007(log),3009(readproc) context=u:r:shell:s0
console:/ $ id log 
uid=1007(log) gid=1007(log) groups=1007(log) context=u:r:shell:s0
```

```
console:/sys/fs # ls -Zl                                                       
total 0
drwxrwxrwt  2 root   root u:object_r:fs_bpf:s0         0 2021-07-20 23:28 bpf
dr-xr-xr-x  2 root   root u:object_r:sysfs:s0          0 2021-07-20 23:51 cgroup
drwxr-xr-x 10 root   root u:object_r:sysfs:s0          0 2021-07-20 23:51 ext4
drwxr-xr-x  3 root   root u:object_r:sysfs_fs_f2fs:s0  0 2021-07-20 23:28 f2fs
drwxr-xr-x  3 root   root u:object_r:sysfs:s0          0 2021-07-20 23:28 fuse
drwxr-xr-x  3 root   root u:object_r:sysfs:s0          0 2021-07-20 23:51 incremental-fs
dr-xr-x---  2 system log  u:object_r:pstorefs:s0       0 2021-07-20 23:28 pstore
drwxr-xr-x  8 root   root u:object_r:selinuxfs:s0      0 1970-01-01 08:00 selinux
```

这里说一下，系统开机后，selinux会把文件系统挂在到/sys/fs/selinux这个节点下面，里面有所有android所定义的对象以及相应的权限控制,部分列举如下

```
console:/sys/fs/selinux/class # ls -l
total 0
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 binder
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 blk_file
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 bluetooth_socket
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 capability
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 capability2
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 chr_file
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 dir
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 fd
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 fifo_file
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 file
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 filesystem
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 hwservice_manage
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 ipc
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 keystore_key
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 lnk_file
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 packet_socket
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 process
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 process2
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 property_service
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 service_manager
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 sock_file
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 socket
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 system
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 tcp_socket
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 udp_socket
dr-xr-xr-x 3 root root 0 2021-07-20 23:28 unix_stream_socket
```

```
console:/sys/fs/selinux/class/binder/perms # ls -lZ
total 0
-r--r--r-- 1 root root u:object_r:selinuxfs:s0  0 2021-07-20 23:28 call
-r--r--r-- 1 root root u:object_r:selinuxfs:s0  0 2021-07-20 23:28 impersonate
-r--r--r-- 1 root root u:object_r:selinuxfs:s0  0 2021-07-20 23:28 set_context_mgr
-r--r--r-- 1 root root u:object_r:selinuxfs:s0  0 2021-07-20 23:28 transfer
console:/sys/fs/selinux/class/binder/perms # 
```

如上可以看到binder所有的权限，可以与其它进程进行binder ipc通信（call），能够向这些进程传递Binder对象（transfer），以及将自己设置为Binder上下文管理器（set_context_mgr）

具体可以查看每个class里面 perms文件夹内的内容。

### SEAndroid 实战1 Service

![seandroid-service](/img/selinux-seandroid/seandroid-service.png)

对于在 Android 6.x （Marshmallow） 之后添加的 Service，如果缺少相关文件或编写不正确，则 开机时service是无法正常启动和运行的。

图中定义了一个mytest_service的服务，声明了一个mytest.rc文件，这样放入`system/etc/init/mytest.rc`里面，那么开机后是无法启动的，原因就是**缺少安全策略**

那么如何定义安全策略呢？

![seandroid-service-ok](/img/selinux-seandroid/seandroid-service-ok.png)

```
//一共需要如下这些文件
mytest_service
mytest.rc
file_context
```

想开机运行服务，需要将相关内容添加到与服务相关的安全标签中（图中file_contexts 和mytest.te）即可

但是这种开机起来的服务是不能注册到binder里面的，也就是不能作为binder服务

那么如何作为binder服务并开机注册呢？请看下图

![seandroid-service-binder](/img/selinux-seandroid/seandroid-service-binder.png)

```
//一共需要如下这些文件
mytest_service
mytest.rc

file_context
service_context.te
service.te
```

### SEAndroid 实战2 Property

Google在Android O以后,为了降低vendor和system之间的耦合度,对property的作用区域也做了明确的区分,分为vendor的property和system里的property.

一般我们自定义property的时候，OEM厂家都应该以`vendor.`开头，或者persist,ro这种的，而不能是xx.yy.zz，否则谷歌认证会报错

![property](/img/selinux-seandroid/seandroid-property.png)

```
//一共需要如下这些文件
mytest.te //主要是声明权限控制的
property_contexts //主要是定义你的某个prop的完整标签上下文
property.te //定义一个新标签类型type
```

这样，你的property，开机后就会能读取或者写入等操作，

也可以使用`getprop -Z `来看你定义的prop的标签内容

### SEAndroid 实战3 Device设备节点

项目中有可能需要自定义了一个设备节点，比如`/dev/test_dev` ，然后要求访问特定设备节点，

那么在SEAndroid中也需要设置相关文件

同时如果你发现系统内某个设备节点无法访问，权限不足，那么你也有能需要重新修改下这个设备节点的SELinux标签，满足权限要求

下面介绍下

![device](/img/selinux-seandroid/seandroid-device.png)

比如一个服务mytest_service想要打开一个设备节点，如上图，那么一般会设计到如下设置

![device-context](/img/selinux-seandroid/seandroid-device-context.png)

```
涉及到设备相关的文件
device.te //主要是定义一个设备节点的type
file_contexts //给具体的设备节点路径定义一个完整的标签，标签中加入你新定义的type
mytest.te //最后在你的服务的域中，定义相关的权限即可
```

当然，如果系统内已经定义好了相关设备节点的标签，那么你直接修改为你自定义的，然后权限自然就有了哦

查看设备节点的标签命令`ls -Z` 因为设备节点属于资源，不是进程

### SEAndroid 实战4  Binder/HIDL

在Android 8.0以后，system和vendor进行了隔离，那么就引入了HIDL来进行通信，其实也是binder通信，

这里总结下Android中的三种binder，以及涉及到的SELinux文件，大家具体可以查看自己家平台里面相关的内容

```
//System binder service相关
file_contexts
service.te
service_context
servicemanager.te 

//HIDL binder service相关
file_contexts
hwservice.te
hwservice_context
hwservicemanager.te 

//vendor中的binder service相关
file_contexts
vndservice
vndservice_contexts 
vndservicemanager.te  
```

### MLS介绍

当大家查看进程标签时`ps -AZ`，一般会发现如下的标签中有个c512,c768,这种的

```
u:r:platform_app:s0:c512,c768  u0_a58     747    287 1140388 136964 SyS_epoll_wait      0 S com.android.systemui
```

SEAndroid里，只定义了s0一个敏感度sensitive，但是定义了0~1023个category。在敏感度只有一个值的情况下，其实MLS已经变成了MCS（Multi-category Security），多组安全。MCS用于隔离，阻断不同组之间的信息流动。颗粒度更细

相关源码定义：

`system/sepolicy/private/mls`     主要策略控制位置

![mls-level](/img/selinux-seandroid/seandroid-mls-level.png)

如上 小写`l`代表level，小写`t` 代表type，

`l1`表示subject的MLS level，`l2`表示object的MLS level

`t1`表示subject的type，`t2`表示object的type

上图的mlsconstrain规则，定义了只有在满足下面四个条件其中之一的情况下，才能对系统的任何目录和文件拥有写、追加、重命名等权限：

（1）dir/file的类型为app_data_file（t2 == app_data_file，t2表示object的类型）

（2）主体和客体的MLS相等（l1 eq l2，l1表示subject的MLS level，l2表示object的MLS level）

（3）主体拥有mlstrustedsubject属性（t1 == mlstrustedsubject ）

（4）客体拥有mlstrustedobject属性（t2 == mlstrustedsubject ）mlstrustedsubject和mlstrustedobject分别是subject和object的属性，相应的类型关联到这两种属性后，可以绕过MLS的限制。

`system/sepolicy/private/mls_decl` 使用了宏生成sensitive和category

`system/sepolicy/private/mls_macros`

`/system/sepolicy/private/seapp_contexts` 文件中的levelFrom字段决定应用和目录/文件的level

`external/selinux/libselinux/src/android/android_platform.c` 文件中，通过对levelFrom的值的判断，赋予应用、应用的数据对应level。

![mls-levelfrom](/img/selinux-seandroid/seandroid-mls-levelfrom.png)

![levelcode](/img/selinux-seandroid/seandroid-mls-levelcode.png)

### Code & Doc

主要代码和文档汇总

谷歌在线文档：[https://source.android.google.cn/security/selinux](https://source.android.google.cn/security/selinux)


源码路径 ：`/system/sepolicy`

其他在线文档，排名不分先后

[https://jung-max.github.io/2019/09/16/Android-SEAndroid%EC%A0%81%EC%9A%A9/](https://jung-max.github.io/2019/09/16/Android-SEAndroid%EC%A0%81%EC%9A%A9/)

[https://milestone-of-se.nesuke.com/en/sv-advanced/selinux/selinux-summary/](https://milestone-of-se.nesuke.com/en/sv-advanced/selinux/selinux-summary/)

https://blog.csdn.net/innost/article/details/19299937/

[https://blog.csdn.net/luoshengyang/article/details/35392905](https://blog.csdn.net/luoshengyang/article/details/35392905)

http://androidxref.com/

[https://www.jianshu.com/p/3f6006e74821](https://www.jianshu.com/p/3f6006e74821)

感谢!

全文 完！
