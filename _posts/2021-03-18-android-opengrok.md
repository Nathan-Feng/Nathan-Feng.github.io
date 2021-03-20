---
layout:     post
title:      "Ubuntu18.04下搭OpenGrok阅读Android 11.0源码"
subtitle:   "阅读，跟踪，进阶源码必备"
date:   2021-03-18 20:56:00 +0800
author:     "Nathan"
tags:

    - OpenGrok
	- Android
    - Tool

---

> 我命由我不由天！
>
> You'll remember you don't believe in any of this fate crap. You're in control of your own life.



### 前言

平时进行ROM开发，查找源码都是直接在公司云内进行`grep`检索，不仅搜索慢，也无法跳转，效率太低。偶然发现`OpenGrok`神器，所以就照着网上的相关教程，成功搭建了`OpenGrok+AOSP 11.0`进行浏览源码，并分享给了其他同事们，下面把相关步骤总结记录，希望对大家有帮助。

### 主机环境：

* Ubuntu 18.04 ，8GB + 1TB
* `Tomcat8`  + `universal-ctags`
* `OpenGrok 1.3.11`
* Android 11.0源码

### 步骤1：下载Android 11.0源码

根据官网的步骤：https://source.android.google.cn/setup/develop#installing-repo 先安装`repo`

![repo](/img/opengrok-android/repo.png)

然后进行源码下载：https://source.android.google.cn/setup/build/downloading#initializing-a-repo-client

我这里下载的是`repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r3`

![git-log](/img/opengrok-android/git-log.png)

由于公司有海外专线，所以没有走国内的代理，大家可以找些国内镜像代理网站。

整个源码下载之后，大约占用103GB左右，

![aosp-volume](/img/opengrok-android/aosp-volume.png)

### 步骤2：软件安装

 * 安装Tomcat8  

   ```
   $ sudo apt-get install tomcat8
   # 有关使用的几个命令
   $sudo service tomcat8 restart //重启
   $sudo service tomcat8 start //启动
   $sudo service tomcat8 stop  //停止
   ```

   测试是否安装成功
   在浏览器输入http://localhost:8080/看到:**It works** 就表示成功了

 * 安装`universal-ctags`

   ```
   sudo apt-get update
   # 卸载 exuberant-ctags
   sudo apt-get remove --purge exuberant-ctags
   # 安装依赖软件 autoconf  
   sudo apt-get install autoconf
   mkdir ~/ctags
   cd ~/ctags
   # 下载universal-ctags，直接从官网下载
   git clone https://github.com/universal-ctags/ctags
   cd ctags
   ./autogen.sh
   # 安装路径。后面索引会用到(/home/xxx/ctags/bin) 其中xxx是ubuntu主机名
   ./configure --prefix=/home/xxx/ctags  
   make -j8
   # 安装（这里会将软件安装到/home/xxx/ctags/bin目录下，这个路径需要在后面写入配置文件中）
   sudo make install
   ```

   ![ctag-install](/img/opengrok-android/ctag-install.png)

 * 安装`OpenGrok`

   官网主页地址https://github.com/oracle/opengrok/releases/

   下载后缀为.tar.gz的文件，不要下载源码，我这里下载的是1.3.11版本(https://github.com/oracle/opengrok/releases/tag/1.3.11)。

   ![opengrok-version](/img/opengrok-android/opengrok-version.png)

   下载后，将它解压到你的工作目录，比如 `~/opengrok/`下。

   然后重命名一下，方便后续使用`mv opengrok-1-3-11 opengrok`

   ![opengrok-all](/img/opengrok-android/opengrok-all.png)

### 步骤3：软件配置

* 多项目配置

  为了便于后续兼容多个项目源码索引，我在opengrok根目录建立了一个database文件夹，专门用于存放各种源码和配置文件的，

  比如database/project1 或者database/project2,这里我先建立一个project1:Android_11.0.0_r3,并建立三个子文件夹`src data etc`,其中`src`里面存放的就是真正的源码，或者是源码的软连接，`data`里面存放的是opengrok执行索引后自动保存的数据，`etc`里面保存了一个配置文件，是用于给tomcat8配置的。

  ![database](/img/opengrok-android/database.png)

  如下是src里面存放的源码软连接,我源码是下载到另一个目录的。所以直接执行`ln -s ~/source_code/aosp11 android `

  ![opengrok-src](/img/opengrok-android/opengrok-src.png)

* 配置Tomcat8

  复制source.war到tomcat中

  ```
  #拷贝完成后，war会被自动解压
  sudo cp ~/soft/OpenGrok/opengrok/lib/source.war /var/lib/tomcat8/webapps/
  ```

  ![webapp-aosp](/img/opengrok-android/webapp-aosp.png)

  然后修改 /var/lib/tomcat8/webapps/source/WEB-INF/web.xml 中 configuration.xml 文件中的`<param-value>`

  ![web-config](/img/opengrok-android/web-config.png)

### 步骤4：构建索引

新建一个openGrok.sh脚本，利用java进行构建：(请根据xxx，替换成你自己的ubuntu路径和名字）

```
java \
    -Djava.util.logging.config.file=/home/xxx/openGrok/opengrok/doc/logging.properties \
    -Xmx4g -jar /home/xxx/opengrok/opengrok/lib/opengrok.jar \
    -c /home/xxx/ctags/bin/ctags \
    -s /home/xxx/opengrok/database/Android_11.0.0_r3/src/android -d /home/xxx/opengrok/database/Android_11.0.0_r3/data -H -P -S -G \
    -W /home/xxx/opengrok/database/Android_11.0.0_r3/etc/configuration.xml -U http://localhost:8080/source \
    -T 2 \
    -m 1024
```

修改脚本的权限 `sudo chmod a+x openGrok.sh`
执行脚本`./openGrok.sh` ,执行过程后会有WARNING和ERROR的打印，直接忽略就好，看内容是由于有些代码是软链接，或者打不开，或者格式不支持引起的，不影响结果。

### 步骤5：验证

构建完成后，直接在浏览器中 **http://localhost:8080/aosp11/**，可以看到如下的效果那表示你就成功了（如果不成功就重启下tomcat8再试一次）

![source-main](/img/opengrok-android/source-main.png)



全文 完！

**下一篇，会介绍：利用nps，进行内网穿透，反向代理。**





