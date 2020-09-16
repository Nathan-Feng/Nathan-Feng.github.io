---
layout:     post
title:      "Android  adb tools "
subtitle:   "Android  adb tools for auto connect by win bat"
date:   2020-09-16 21:41:00 +0800
author:     "Nathan"
tags:

    - Tools

---

> 想知道如何改变世界吗？一次只做好一件小事！
>
> You wanna know how to change the world? One act of random kindness at a time.
>



### 前言

笔者在平时工作中，经常会使用`adb` 命令来进行Android开发。不过在开发前总是要先连接板子才行，但是过程会稍微有些繁琐，所以有了本文的内容。

开发环境：

* Win10 + Android Studio
* 串口 + CMD窗口
* 开发板 
* B网卡 + 网口

在开发过程中，一般的过程如下：

- 板子开机进入launcher
- 通过串口看板子获取的IP地址(`ifconfig`，通过B网卡一般为`192.168.137`段的) 
- Win10 打开一个`cmd`窗口
- 输入`adb connect xxx.xxx.xxx.xxx`
- 输入`adb remount`
- 输入`adb push xxx xxxx`

更麻烦的是，每次reboot之后，板端的IP地址就变了。所以需要重复上述过程，非常耗时而且繁琐

## 神器：adb 一键连接登场

针对上述痛点，笔者想：能不能不用每次查看板端ip，然后使用批处理自动找到ip并连接呢？

思路如下：

1. 板子开机后会自动获取某一个ip
2. 电脑端查询到此时所有的ip地址并过滤出来
3. 然后根据查询到的ip地址进行adb connect自动操作

笔者想到了BAT脚本的功能，并赶鸭子上架，学习了几个实用的命令 `arp`，问题解决了。

话不多说，直接上脚本：

**`adb.bat`**

```JAVA
@echo off&setlocal enabledelayedexpansion
echo 尝试连接开始,请稍候……
arp -d >nul
adb disconnect >nul
for /f %%a in ('arp -a^|findstr "192.168.137"') do (
	::echo %%a
	adb connect %%a > nul 2> nul
	adb root > nul 2> nul
	adb remount > nul 2> nul > tmp.tmp
	for /f %%j in (tmp.tmp) do (
		set enum=%%j
		if "%%j"=="remount" (
			del /a /q *.tmp
			echo 连接 %%a 成功 &pause
			goto end
		) else (
			del /a /q *.tmp
			echo 连接失败请检查环境 &pause
		) 
	)
)

:end
start cmd 

```

简单分析如上脚本：

`arp -d` 对arp缓存信息的删除 ，因为笔者的板子在每次重启的时候都会重新获取IP，导致上次的ip地址被缓存，所以执行这个命令可以删除过期无效的IP。不过这个命令需要右键管理员权限才能运行，否则会提示如下内容

![arp-d](/img/adb-tool/arp-d.png)

![admin](/img/adb-tool/admin.png)

`adb disconnect` 由于每次重启时，adb devices都会记录上一次的设备信息，所以也清理下所有连接设备。

`arp -a` 列出所有局域网中所有的IP,如下图中可以看到B网卡所有的ip地址，其中**192.168.137.41**就是板子的地址。

然后加上过滤`|findstr  "192.168.137"` 就可以找到所有以192.168.137开始的结果了。

![arp-a](/img/adb-tool/arp-a.png)

剩下就是根据过滤的结果进行操作了，比较简单，

需要注意的是，过滤结果会把 上图中的**接口：192.168.137.1 --- 0xb** 也会过滤出来，所以循环会进行两遍。不过由于时间较短，就未做优化。

然后就是把`adb remount` 的结果输出到了一个临时文件中，然后读取这个文件的结果来判断是否执行成功


最后放上执行的过程：

![done](/img/adb-tool/done.png)

## 总结

- 从有想法到实现，也就半天的时间(现学bat脚本现卖)

- 希望本文对大家有所帮助，能够对您在解决工作和生活中的一些小痛点时有所启发

- 本文工具下载地址： [传送门](https://github.com/Nathan-Feng/Tools)

  