---
layout:     post
title:      " 让app的内容可供谷歌ATV Launcher(语音/文字)搜索"
subtitle:   " Make your apps content searchable when using voice on the ATV home screen "
date:   2019-12-12 14:00:00
author:     "Nathan"
tags:
    - ATV
---



### 前言

---

上回写到了让app的内容可以主动推送到launcher上去显示以及交互，而本文说下如何让用户在谷歌TV launcher上进行语音和文字搜索时，能显示app中提供的内容。

总结下步骤：

1. 构建本地数据库xx.db
2. 增删改查你的xx.db，
3. 利用FileProvider，共享你的xx.db给谷歌的launcher
4. 处理谷歌launcher语音或者文字搜索时传进来的intent

[官方传送门](https://developer.android.google.cn/training/tv/discovery/searchable#top_of_page)

本文基于官方例子而总结的，[传送门](https://github.com/android/tv-samples/tree/master/Leanback)

涉及到的核心类：

```java
android.app.SearchManager.java
android.provider.BaseColumns.java    
```



## 1.构建本地数据库

利用Sqlite创建一个本地数据库，数据库的名字和table名字都随便取，主要是表的相关列需要注意下：

必选列有两个：

- [_ID](https://developer.android.google.cn/reference/android/provider/BaseColumns.html#_ID)
- [SUGGEST_COLUMN_TEXT_1](https://developer.android.google.cn/reference/android/app/SearchManager.html#SUGGEST_COLUMN_TEXT_1)

其他可选列，请参考[SearchManager](https://developer.android.google.cn/guide/topics/search/adding-custom-suggestions.html#SuggestionTable)

### 2.复写增删改查

此处省略

### 2.共享你的`ContentProvider`

在AndroidManifest.xml中声明如下：

```java
<provider
  android:name=".provider.data.VideoProvider"
  android:authorities="com.example.tvrecommend"//唯一标识
  android:permission="com.example.tvrecommend.ACCESS_VIDEO_DATA"
  android:exported="true">//必须声明为true
  <path-permission
   android:pathPrefix="/search"//一般为这个
   android:readPermission="android.permission.GLOBAL_SEARCH" />//必须是这个权限
</provider>
```

然后在res/xml下建立一个[searchable.xml](https://developer.android.google.cn/guide/topics/search/searchable-config.html)的文件

![image-20191211163613859](/img/atv-search/searchable.png)

着重介绍下这个文件：

```java
<searchable xmlns:android="http://schemas.android.com/apk/res/android"
    android:hint="@string/search_hint"//当搜索时提供一些提示
    android:includeInGlobalSearch="true"//true时，在Settings->Device Preferences->Google->Searchable app中的开关会打开，也就是让谷歌默认搜索这个app,图片如下
    android:label="@string/search_label"//需要跟AndroidManifest.xml中的一样
    android:icon="@drawable/ic_launcher"//图标显示
    android:searchSettingsDescription="@string/settings_description"
    android:searchSuggestAuthority="com.example.tvrecommend"//需要跟AndroidManifest.xml中provider的一样才行。
    android:searchSuggestIntentAction="android.intent.action.VIEW"//默认
    android:searchSuggestIntentData="content://com.example.tvrecommend/dvbchannel"//当搜索显示结果后，如果用户点击时，会触发回传这个数据给用户，然后再外加一些额外ID数据的。
    android:queryAfterZeroResults="true"
    android:searchSuggestPath="search"//需要跟AndroidManifest.xml中provider的一样才行。
    android:searchSuggestSelection=" ?"
    android:searchSuggestThreshold="0"//最小查询的门限，默认是0
    android:voiceSearchMode="showVoiceSearchButton|launchRecognizer"
    android:voiceLanguageModel="free_form"
    android:voicePromptText="@string/search_invoke"
    android:voiceLanguage="@string/language"/>
```

![image-20191211170104481](/img/atv-search/searchable-app.png)

我这里的数据库插入数据后的样子如下：

![image-20191211172445556](/img/atv-search/db-all.png)

其中table 名字是dvb,说明一下主要列：

- `channel_id`：是我用于播放节目的唯一标识,自定义的
- `suggest_text_1(SearchManager.SUGGEST_COLUMN_TEXT_1)`
- `suggest_test_2(SearchManager.SUGGEST_COLUMN_TEXT_2)`这两个是用于谷歌搜索时提供关键字的，只要满足任意一个字，就能显示出来
- `suggest_result_card_image(SearchManager.SUGGEST_COLUMN_RESULT_CARD_IMAGE)`:用于显示搜索结果时的背景图片。
- 另外还有一列：`suggest_intent_action(SearchManager.SUGGEST_COLUMN_INTENT_ACTION)`，用于用户点击搜索结果时的回传标识

当搜索结束时，识别的语音或者文字会调用你的`query`()方法

其中rawQuery就是识别的文字。然后再根据数据库中匹配相应的文字来找到相应的结果。

![image-20191211174708683](/img/atv-search/query-word.png)

弄个git图看下效果：

![search](/img/atv-search/search.gif)



### 3.处理用户交互

在AndroidManifest.xml中主要注册一个activity来接受处理来自launcher交互传递过来的intent数据

```java
		<meta-data android:name="android.app.default_searchable"//固定值
            android:value=".provider.VideoDetailsActivity" />//需要接受intent的activity
        <activity
            android:name=".provider.VideoDetailsActivity"//同上
            android:exported="true">

            <!-- Receives the search request. -->
            <intent-filter>
                <action android:name="android.intent.action.SEARCH" />//必须过滤这个
            </intent-filter>

            <!-- Points to searchable meta data. -->
            <meta-data//指向需要的配置文件
                android:name="android.app.searchable"
                android:resource="@xml/searchable" />
        </activity>
```

当用户点击搜索的结果时，相应的activity就会接受到相应的intent

```java
D zyf     : VideoDetailsFragment:onCreate:
D zyf     : VideoDetailsFragment:hasGlobalSearchIntent: intentAction:GLOBALSEARCH
D zyf     : ideoDetailsFragment:intentData:content://com.example.tvrecommend/dvbchannel/20190
```

为了测试本地推送图片显示，找了一个工具类：`CopyUtil.java`，用于把assets资源中的图片拷贝到本地供launcher读取。相关代码请查询demo.另外图片必须`grantUriPermission`后，launcher才有权限读取和显示的。



## 总结：

- 搜索内容本质上就是app提供相应的数据源(xx.db)，然后让launcher跨进程调用
- 另外就是根据intent处理交互的逻辑了。
- 如果想显示本地共享的图片，请注意，`android:sharedUserId="android.uid.system"`就不能加了
- [demo传送门](https://github.com/Nathan-Feng/Android-TV-Recommend-Demo)，如果觉得有用，欢迎给个star，谢谢。