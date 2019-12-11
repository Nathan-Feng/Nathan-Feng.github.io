---
layout:     post
title:      " 如何在谷歌Android TV Launcher上显示自定义的Channel/Program"
subtitle:   " 基于java "
author:     "Nathan"
tags:
    - ATV
---

> 如何在谷歌Android TV Launcher上显示自定义的ChannelProgram（基于java）
### 前言

---

本人最近在做DVB的项目，需要把DVB里面的节目推送到谷歌的launcher上去显示，更新，以及与用户进行交互，以下记录一些主要方法和步骤。

[官方参考地址](https://developer.android.google.cn/training/tv/discovery/recommendations-channel#java) （默认已经是中文了）

![image-20191210173234486](/img/atv-channel/home-launcher.png)



步骤只要分三步
1. 准备前提条件
2. 创建Channel以及发布
3. 创建program
4. 处理交互



### 前提条件准备

```java
//AndroidManifest.xml中添加权限
<uses-permission android:name="com.android.providers.tv.permission.WRITE_EPG_DATA"/>

//build.gradle 中添加依赖：
implementation 'androidx.tvprovider:tvprovider:1.0.0'
```

涉及到的类如下：

```java
androidx.tvprovider.media.tv.TvContractCompat.java
androidx.tvprovider.media.tv.Channel.java
androidx.tvprovider.media.tv.ChannelLogoUtils.java
androidx.tvprovider.media.tv.PreviewProgram.java
```



### 关于Channel说明

先拿Youtube作为例子，了解下大致内容，

![image-20191210112508418](/img/atv-channel/youtube.png)

如上图，分四个部分，

- 左边是应用的**图标**和**名称**，
- 上面是Channel的名称：`Recommended`
- 中间是一个卡片背景，以及一个节目时长
- 下面是节目的名称，详情等信息

根据[官方文档](https://developer.android.google.cn/training/tv/discovery/recommendations-channel#java)，Android TV 主屏幕使用 Android 的 `TvProvider` API 来管理您的应用创建的频道和节目，谷歌的ATV Launcher会默认读取`data/data/com.android.providers.tv/databases/tv.db`

![image-20191210111035493](/img/atv-channel/tv-db.png)

大家可以拷贝出来，使用`SQLite Expert`软件来打开查看里面内容，

![image-20191210113515252](/img/atv-channel/tv-db.png)

其中`channels`主要就是我们创建channel后，保存channel信息的地方了，

![image-20191210114826596](/img/atv-channel/sqlite-channel.png)

点开后，我们看第4和5行，也就是package name是youtube的那两行，

1. 首先看下`type`,都是`TYPE_PREVIEW` ，意思就是只又channels中的type符合TYPE_PREVIEW，launcher才会认为是一个推送channel，才会在launcher上显示。然后才会去`preview_programs`表中查找相应`_id`，所以能解释为什么[官网](https://developer.android.google.cn/training/tv/discovery/recommendations-channel#creating_a_channel)说的：频道类型必须是 `TYPE_PREVIEW`
2. 再看下`display_name`这一列，也就是显示在主页中每一行的标题
3. 另外还有logo，存储的就是logo图片了



看下如何构建Channel：

步骤1：

```java
Channel channel = new Channel.Builder()
                .setDisplayName(mediaChannel.getName())//设置dispaly_name那一栏
                .setDescription(mediaChannel.getDescription())//设置description那一栏
                .setType(TvContractCompat.Channels.TYPE_PREVIEW)//注意：必须写死
                .setInputId(channelInputId)//也就是_id这一栏,用于以后查询，更新时使用
                .setAppLinkIntentUri(Uri.parse(SCHEME + "://" + APPS_LAUNCH_HOST
                        + "/" + START_APP_ACTION_PATH))//设置app_link_intent_uri 
                .setInternalProviderId(mediaChannel.getMediaChannelId())                				.setAppLinkColor(context.getResources().getColor(R.color.colorAccent))
                .build();
```

步骤2：将频道插入提供程序：

```java
 Uri channelUri = context.getContentResolver().insert(
            TvContractCompat.Channels.CONTENT_URI, builder.build().toContentValues());
```

步骤3：保存channelId

```java
long channelId = ContentUris.parseId(channelUri);
```





**Channel 部分总结一下：**

`channels`这个表中需要关注几个选项有：

	- package_name ：应用的包名
	- type：默认都一样
	- display_name ：显示Channel名称
	- app_link_intent_uri :当点击左边应用图标时会启动一个intent，打开应用
	- logo ：应用的图标

### 关于Program说明

书接上回：再看`preview_programs`这个表：顾名思义，就是预览program的地方,

![image-20191210150524322](/img/atv-channel/sqlite-program.png)

如上图(截取部分)，主要关于几个属性如下：

- `_id` :这个值跟channels表中的`id`是对应的，也就是`channel`表中的`_id`可能对应多个`preview_program`表中的多个program，也就是一对多。

- `title` :显示的节目的名称

- `short_description` ：关于节目的详细信息

- `poster_art_uri` :就是海报显示的图片

- `duration_millis` ：显示的时长，默认是ms毫秒级的，launcher会自动转成时分秒的。

- `intent_uri` : 当点击某个节目时，会触发一个intent给接受者，接受者可以根据这个intent来启动播放某个节目

- `logo_uri` :也就是显示一个小图标，

- `interaction_count` ：显示的点击数，如图中的28,404,523 Views

  

看下如何构建Program：

步骤1：构建PreviewProgram

```java
PreviewProgram program = new PreviewProgram.Builder()
          .setChannelId(channelId)//设置影响_id这一栏
          .setTitle(mp.getTitle())//设置titile节目名称
          .setDescription(mp.getDescription())//设置short_description 
          .setPosterArtUri(Uri.parse(mp.getCardImageUrl()))//设置预览图片poster_art_uri
          .setIntentUri(Uri.parse(SCHEME + "://" + APPS_LAUNCH_HOST
                            + "/" + PLAY_MEDIA_ACTION_PATH + "/" + mediaProgramId))
           .setPreviewVideoUri(Uri.parse(xxx))//设置预览动画，preview_video_uri
           .setInternalProviderId(mediaProgramId)
           .setContentId(contentId)
           .setWeight(weight)
           .setType(TvContractCompat.PreviewPrograms.TYPE_TV_SERIES)//设置节目类型，详情看官网介绍
           .setAuthor("Author")//设置作者
           .setDurationMillis(30000)//设置时长
           .build();
```

步骤2：插入节目

```java
 Uri programUri = context.getContentResolver().insert(PREVIEW_PROGRAMS_CONTENT_URI,
                    program.toContentValues());
```

步骤3：保存programId

```java
long programId = ContentUris.parseId(programUri);
```



### 关于Channel显示和发布

创建channel后，launcher中如果想立即显示，有几个方法：*

创建默认Channel时，调用 `TvContractCompat.requestChannelBrowsable(context, channelId);` 其中channelId就是创建channel时返回的id，此时如果preview_programs表里面存在_id与channelId相同时，那么就会都显示在launcher上。

**注意**：如果你的apk是系统级的，AndroidManifest.xml中加入`android:sharedUserId="android.uid.system"`，那么及时你是创建的默认channel，也是不会立即显示在launcher上的(*这个问题已经反映给谷歌了，应该是个bug*)。那么想显示的话，那需要如下方法:

在你的apk内，调用如下方法，弹窗让用户选择

```java
Intent intent = new Intent(TvContractCompat.ACTION_REQUEST_CHANNEL_BROWSABLE);
        intent.putExtra(TvContractCompat.EXTRA_CHANNEL_ID, channelId);
        try {
            this.startActivityForResult(intent, MAKE_BROWSABLE_REQUEST_CODE);
        } catch (ActivityNotFoundException e) {
            Log.e(TAG, "Could not start activity: " + intent.getAction(), e);
        }
```

这样就会显示如下图片

![image-20191210155415519](/img/atv-channel/channel-prompt.png)

当用户选择add后，就能立即显示了(其实原理就是让系统强行设置tv.db中channel表中相应channelId的`browsable`属性设置为1)

另一个方法就是在ATV的launcher中最下方有个选择：

![image-20191210160120303](/img/atv-channel/customize-channels.png)

点击进去后，选择相应的channel，Open/Close即可。



### 关于节目预览图片

有些时候需要离线显示推送的节目，这时候就需要默认显示一些本地图片，

**步骤1**：共享app的路径：

在AndroidManifest.xml中声明FileProvider,关于具体用法，可自行百度

```java
<provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.example.tvrecommend.fileprovider"
            android:grantUriPermissions="true"
            android:exported="false">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
</provider>
```

**步骤2**：需要注意的是，我这边测试谷歌的launcher，在ATV 8.0以上的平台上，没有读取sdcard上文件夹目录的权限，很是郁闷。

![image-20191210171325546](/img/atv-channel/sdcard.png)

但是app的私有目录是可以的，比如data/data/你的包名（也就是files-path），或者sdcard/Android/data/你的包名也就是external-files-path）是有共享权限的。

注意：系统级app，也就是上文提到的`android:sharedUserId="android.uid.system"`是无法提供所有本地app的。个人觉得，谷歌的launcher由于没有平台签名，所以无法访问特定功能。

**步骤3**：共享具体文件

```java
Uri uri = FileProvider.getUriForFile(this, "com.example.tvrecommend.fileprovider", file);
grantUriPermission("com.google.android.tvlauncher", uri, Intent.FLAG_GRANT_READ_URI_PERMISSION
                | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
```

**步骤4**：把步骤3得到的uri，设置给`PreviewProgram`的`setPosterArtUri`即可



其他还有用户交互部分，直接看[官网介绍](https://developer.android.google.cn/training/tv/discovery/recommendations-channel#android_tv_home_screen_events)即可

## 总结

 - 推送节目主要就是通过content provider操作tv.db，另外谷歌已经封装好很多接口，直接用就行。

 - 多查看官网的资料，很多细节都在里面。

 - 本文demo地址：[传送门](https://github.com/Nathan-Feng/Android-TV-Recommend-Demo) 如果觉得有用，欢迎给个**star**，谢谢

   

   

