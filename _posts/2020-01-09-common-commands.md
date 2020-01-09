---
layout:     post
title:      "Android开发常用命令总结"
subtitle:   "Android Development useful commands summary"
date:   2020-01-09 09:51:00 +0800
author:     "Nathan"
tags:
    - Tools

---

> 幸运是机会加上准备
>
> Luck is where opportunity meets preparation
>

## 前言

在Android开发过程中，经常会使用一些命令来帮助自己调试或者增加开发效率，如下是常用的命令总结，以后会不定时的更新。

### 1.ADB常用命令

* `adb connect your IP` // 可能会要求添加端口号，默认5555
* `adb disconnect` //断开所有已连接的设备
* `adb shell` //进入指定设备
* `adb -s serialNumber shell` //进入指定设备
* `adb logcat` //查看日志
* `adb devices` //查看设备
* `adb get-state` //连接状态
* `adb start-server` //启动ADB服务
* `adb kill-server` //停止ADB服务
* `adb push local remote` //电脑推送到设备
* `adb pull remote local` //设备拉取到电脑
* `adb reboot` //重启设备
* `adb reboot bootloader` //重启到bootloader刷机模式
* `adb reboot recovery` //重启到recovery模式
* `adb install -r <apkfile>`  //保留数据和缓存文件，重新安装apk：
* `adb install -s <apkfile>` // 安装apk到sd卡
* `adb uninstall <package>`  //卸载APK
* `adb uninstall -k <package>` // 卸载app但保留数据和缓存文件：

### 2.am&pm相关命令

* `am start -n <package_name>/.<activity_class_name>` // 启动应用
* `am kill<packageName> `杀app的进程
* `am force-stop <packageName> ` 强制停止一切
* `am startservice` 启动服务
* `am stopservice` 停止服务
* `am start -a android.intent.action.VIEW -d http://www.12306.cn/ `打开12306网站
* `am start -a android.intent.action.CALL -d tel:10086` 拨打10086
* `am restart` //重启虚拟机，不重启设备
* `pm list packages` 列出设备中所有的包名
* `pm install/uninstall` 安装/卸载

### 3.用户事件输入

* **输入字符**：`adb shell input text "xxx"`
* **输入按键**：`adb shell input keyevent <KEYCODE>` 其中KEYCODE见[链接](https://developer.android.google.cn/reference/android/view/KeyEvent.html?hl=en)，
* **点击事件：** `input tap  ` 例点击坐标（500，500），相应指令： `input tap 500 500`.
* **滑动事件：** `input swipe     ` 例从坐标(300，500)滑动到(100，500)，相应指令： `input swipe 300 500 100 500`. 例200ms时间从坐标(300，500)滑动到(100，500)，相应指令： `input swipe 300 500 100 500 200`.

input的用法：

`console:/ # input
Usage: input [<source>] [-d DISPLAY_ID] <command> [<arg>...]

The sources are: 
      dpad
      keyboard
      mouse
      touchpad
      gamepad
      touchnavigation
      joystick
      touchscreen
      stylus
      trackball

-d: specify the display ID.
      (Default: -1 for key event, 0 for motion event if not specified.)
The commands and default sources are:
      text <string> (Default: touchscreen)
      keyevent [--longpress] <key code number or name> ... (Default: keyboard)
      tap <x> <y> (Default: touchscreen)
      swipe <x1> <y1> <x2> <y2> [duration(ms)] (Default: touchscreen)
      draganddrop <x1> <y1> <x2> <y2> [duration(ms)] (Default: touchscreen)
      press (Default: trackball)
      roll <dx> <dy> (Default: trackball)
      event <DOWN|UP|MOVE> <x> <y> (Default: touchscreen)`

### 4.logcat相关

* `echo 0 > /proc/sys/kernel/printk` //关闭内核打印(0到7)

### 5.find&grep 相关

* grep -rn 'aaa'   --exclude-dir={0000,1111,222}  //排除目录
* logcat | grep -vE "DEBUG|BluetoothManagerService" //反过滤
* find ./ -name "*.mk" | grep -rn "xxx" //在当前目录下所有.mk结尾的文件中搜索xxx字符

### 6.挂载相关

* `mount -o rw,remount /system`  //重新挂载分区使可读写(参数包括/、/system、/vendor等)
* `adb root` -> `adb disable-verity` -> `adb remount`

### 7.Selinux相关

* `setenforce 0`  //临时关闭selinux，打开就是1
* `getenforce`  // ( Permissive/Enforcing )查看当前的模式









