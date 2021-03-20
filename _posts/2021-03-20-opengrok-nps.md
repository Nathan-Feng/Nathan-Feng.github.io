---
layout:     post
title:      "NPS和Opengrok结合进行内网穿透"
subtitle:   "NPS+Opengrok=AOSP Online"
date:   2021-03-20 10:54:00 +0800
author:     "Nathan"
tags:

    - nps
    - Tool

---

> 只有鼓起勇气告别海岸，才能发现新的海洋
>
> You can never cross the ocean unless you have the courage to lose sight of hte shore.



### 前言

上篇利用Opengrok进行安卓源码的检索，在自己电脑浏览倒是没有问题，但是我想共享给其他同事，或者放在网上给网友使用呢，那就需要今天的主角nps上场了。引用官网原话：一款轻量级、高性能、功能强大的内网穿透代理服务器。支持tcp、udp、socks5、http等几乎所有流量转发，可用来访问内网网站、本地支付接口调试、ssh访问、远程桌面，内网dns解析、内网socks5代理等等……，并带有功能强大的web管理端。话不多说，开始教程吧。

官网：[https://github.com/ehang-io/nps](https://github.com/ehang-io/nps)

### 内网穿透介绍

什么是内网穿透？引用[https://blog.csdn.net/hteacher001/article/details/105858724](https://blog.csdn.net/hteacher001/article/details/105858724) 说的很好,我直接引用过来
内网穿透，实际上是三台电脑之间的故事，分别是客户端（用户访问的服务器），中转服务器（云服务器，可被外界访问的服务器）和内网服务器；
总之一句话，内网穿透就是客户端通过访问中转服务器间接性的去访问内网服务器中的东西。

对于我这边，公司电脑都是加域网络，而ubuntu主机无法获取内网ip，所以想通过我电脑作为代理服务器，让其他同事通过我的电脑来访问我的ubuntu服务器中的opengrok，画个图一目了然，如下：

![nps-info](/img/nps/nps-info.png)

### 步骤1：nps工具下载和安装

根据上图中说明，我下载了win10的服务端和linux的客户端工具，下载地址：[https://github.com/ehang-io/nps/releases](https://github.com/ehang-io/nps/releases)

全家福如下图[v0.26.9](https://github.com/ehang-io/nps/releases/tag/v0.26.9)（请忽略android_client.apk）

![nps-tools](/img/nps/nps-tools-all.png)

### 步骤2：软件安装

### 都是按照官网说明的https://github.com/ehang-io/nps/blob/master/README_zh.md

#### 服务端启动

下载完服务器压缩包后，解压，然后进入解压后的文件夹，执行安装命令

对于windows，管理员身份运行cmd，进入安装目录 `nps.exe install`

- 默认端口

  nps默认配置文件使用了80，443，8080，8024端口

  80与443端口为域名解析模式默认端口

  **8080为web管理访问端口**

  **8024为网桥端口，用于客户端与服务器通信**

- 启动

  对于windows，管理员身份运行cmd，进入程序目录 `nps.exe start`

```
安装后windows配置文件位于 C:\Program Files\nps
```

![nps-conf](/img/nps/nps-conf.png)

​	访问服务端ip:web服务端口（默认为8080）

​	使用用户名和密码登陆（默认admin/123，正式使用一定要更改）

在浏览器输入http://localhost:8080/看

![nps-main-page](/img/nps/nps-main-page.png)

登陆的用户名和密码默认是admin/123

### 步骤3：nps配置客户端

登陆后，点击`客户端`->`新增` ->`允许客户端进行配置文件连接`->`否` ，其他默认，最后点击`新增`

![client-new](/img/nps/client-new.png)

完成后，点击`+`号可以查看客户端需要执行的命令行

![client-cmd](/img/nps/client-cmd.png)

### 步骤4：配置nps客户端的隧道

新建**隧道**

![channel-click](/img/nps/channel-click.png)

配置**隧道参数**

![client-channel-my](/img/nps/client-channel-my.png)

填写完毕后的效果如下：

![client-channel](/img/nps/client-channel.png)

### 步骤5：nps客户端启动

在客户端，解压后，直接执行如下命令,看到打印中有successfull字样即可

注意参数中server端的**IP地址**和端口号

![client-start](/img/nps/client-start.png)



感谢如下网址：

[https://blog.csdn.net/hteacher001/article/details/105858724](https://blog.csdn.net/hteacher001/article/details/105858724)

全文 完！







