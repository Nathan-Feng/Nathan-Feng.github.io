---
layout:     post
title:      "树莓派4B结合OpenGrok阅读Android 11.0源码"
subtitle:   "RaspberryPI+Opengrok+AOSP"
date:   2021-04-05 13:09:00 +0800
author:     "Nathan"
tags:

    - OpenGrok
    - Android
    - 树莓派

---

> 穿过失败的迷雾，方能瞥见成功的曙光！
>
> Failure is the fog through which we glimpse triumph



### 前言

前两篇文章，介绍了利用opengrok进行阅读android源码，感觉挺好用的。但是在家时就没法访问公司内网了，突然想起吃灰神器树莓派还躺着，所以就趁机也打造一个`树莓派+opengrok+android源码`，由于我这里只有16GB的sdcard，所以就花了点钱修好了一个1GB的机械移动硬盘，这样也避免再次吃灰。万事具备，那就说干就干吧！

### 主机环境：

* 树莓派4B (4GB RAM + 16GB sdcard) + 1TB 移动硬盘
* `Tomcat10  + universal-ctags + openjdk 11`
* `OpenGrok 1.6.7`
* `android-11.0.0_r3`源码

### 步骤1：树莓派 下载Android 11.0源码

我这边根据清华大学开源镜像网站的：[https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/) 先安装`repo`

下载 repo 工具:

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

如果需要某个特定的 Android 版本([列表](https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds))：

```
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-11.0.0_r3
```

同步源码树（以后只需执行这条命令来同步）：

```
repo sync
```

最后删掉源码中不需要存在的索引目录

```
#删除源代码下面可能存在的".git"隐藏目录
$ find . -name ".git" | xargs  rm -rf
 # 删除无法进行索引的文件
$ find . -name "*.apk" | xargs  rm -rf
$ find . -name "*.zip" | xargs  rm -rf
$ find . -name "*.jar" | xargs  rm -rf
# 编译工具是没有必要的目录
$ rm -rf prebuilts
```



### 步骤2：软件安装

* 安装openjdk11

  运行以下命令安装最新的 JDK 版本，目前是 OpenJDK 11 JDK：

  ```
  sudo apt update
  sudo apt install default-jdk
  ```

  安装完成后，通过命令可以检查 Java 版本进行验证：

  ```
  java -version
  ```

 * 安装Tomcat10  [https://tomcat.apache.org/download-10.cgi](https://tomcat.apache.org/download-10.cgi)

   ![tomcat10](/img/raspberrypi-android/tomcat10.png)

   下载好Tomcat 使用下面解压命令进行解压,我把压缩文件放在/home/pi目录下

   ```bash
   tar -zxf apache-tomcat-10.0.4.tar.gz
   ```

   解压完成后 需要 cd到 tomcat 的bin目录，运行命令

   ```bash
   cd apache-tomcat-10.0.4/bin
   ./startup.sh
   #关闭服务器命令是
   ./shutdown.sh
   ```

   测试是否安装成功,在浏览器输入[http://localhost:8080/](http://localhost:8080/)看到如下就表示成功了

   ![tomcat10-start](/img/raspberrypi-android/tomcat10-start.png)

 * 安装`universal-ctags`

   ```
   sudo apt-get update
   # 卸载 exuberant-ctags
   # 安装依赖软件 autoconf  
   sudo apt-get install autoconf
   mkdir ~/opengrok/ctags
   cd ~/opengrok/ctags
   # 下载universal-ctags，直接从官网下载
   git clone https://github.com/universal-ctags/ctags
   cd ctags
   ./autogen.sh
   # 安装路径。后面索引会用到(/home/pi/opengrok/ctags/ctags/bin)
   ./configure --prefix=/home/pi/opengrok/ctags/ctags  
   make
   # 安装（这里会将软件安装到/home/pi/opengrok/ctags/ctags/bin目录下，这个路径需要在后面写入配置文件中）
   sudo make install
   ```
   

![ctags-install](/img/raspberrypi-android/ctags-install.png)

 * 安装`OpenGrok`

   官网主页地址[https://github.com/oracle/opengrok/releases/](https://github.com/oracle/opengrok/releases/)

   下载后缀为.tar.gz的文件，不要下载源码，我这里下载的是opengrok-1.6.7.tar.gz版本([https://github.com/oracle/opengrok/releases/tag/1.6.7](https://github.com/oracle/opengrok/releases/tag/1.6.7))。

   下载后，将它解压到你的工作目录，比如 `~/opengrok/`下。

   然后重命名一下，方便后续使用`mv opengrok-1.6.7 opengrok`


### 步骤3：软件配置

* 多项目配置

  为了便于后续兼容多个项目源码索引，我在opengrok根目录建立了一个database文件夹，专门用于存放各种源码和配置文件的，

  比如database/project1 或者database/project2,这里我先建立一个project1:Android_11.0.0_r3,并建立三个子文件夹`src data etc`,其中`src`里面存放的就是真正的源码，或者是源码的软连接，`data`里面存放的是opengrok执行索引后自动保存的数据，`etc`里面保存了一个配置文件，是用于给tomcat10配置的。

  ![database](/img/raspberrypi-android/android-database.png)

  由于树莓派sdcard空间很小，所以我src和data下面的数据都是通过软连接到移动硬盘中的，如上图

  

* 配置Tomcat10

  复制source.war到tomcat中

  ```
  #拷贝完成后，war会被自动解压
  sudo cp ~/opengrok/opengrok/lib/source.war /var/lib/tomcat8/webapps/aosp11
  ```

  然后修改 `/home/pi/tomcat/apache-tomcat-10.0.4/webapps/aosp11/WEB-INF/web.xml` 中 configuration.xml 文件中的`<param-value>`

  ![web-config](/img/raspberrypi-android/web-config.png)

  最后需要配置开机自启动，否则就需要手动执行脚本才行
  
  ```
  TODO
  ```
  
  

### 步骤4：构建索引

新建一个build.sh脚本，利用java进行构建：

```
java \
    -Djava.util.logging.config.file=/home/pi/opengrok/opengrok/doc/logging.properties \
    -Xmx2g -jar /home/pi/opengrok/opengrok/lib/opengrok.jar \
    -c /home/pi/opengrok/ctags/ctags/bin/ctags \
    -s /home/pi/opengrok/database/Android_11.0.0_r3/src/android -d /home/pi/opengrok/database/Android_11.0.0_r3/data -H -P -S -G \
    -W /home/pi/opengrok/database/Android_11.0.0_r3/etc/configuration.xml -U http://localhost:8080/aosp11 \
    -T 2 \
    -m 1024
```

![opengrok-start](/img/raspberry-android/opengrok-start.png)

修改脚本的权限 `sudo chmod a+x build.sh`
执行脚本`./build.sh` ,执行过程后会有WARNING和ERROR的打印，直接忽略就好，看内容是由于有些代码是软链接，或者打不开，或者格式不支持引起的，不影响结果。

![opengrok-warning](/img/raspberrypi-android/opengrok-warning.png)

### 步骤5：验证

构建完成后，直接在浏览器中 [**http://localhost:8080/aosp11/**](https://source.android.google.cn/setup/develop#installing-repo)，可以看到如下的效果那表示你就成功了（如果不成功就重启下tomcat10再试一次）

![source-main](/img/opengrok-android/source-main.png)



### 小问题解决

1. 外接移动硬盘NTFS不能读写

   ```
    sudo apt-get install ntfs-3g
    modprobe fuse
    然后umount一下,重启后就好了
   ```

   

2. 建议不要修改/etc/fstab 否则可能会无法开机

   ```
   这个网址里面的工具我都尝试过，不行，还不如直接找个ubuntu电脑，还原下etc/fstab呢
   https://blog.csdn.net/yes169yes123/article/details/105605271
   ```

### 总结

树莓派4B买了差不多两年了，本来想好好玩玩的，无奈真是像网友说的，变成吃灰神器，经过本文调教，也算是没白花冤枉钱，

以后有其他好玩的项目，也会再多玩玩

全文 完！









