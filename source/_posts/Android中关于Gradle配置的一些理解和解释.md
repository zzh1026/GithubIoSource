---
title: Android中关于Gradle配置的一些理解和解释
comments: false
date: 2018-06-01 11:09:37
tags: Android, gradle配置, compileSdk,Platform-tools, classpath配置
categories: 
    - Android
    - 知识总结

---

# 简介 #

本篇文章主要介绍Android的Gradle配置中关于compileSdk、minSdk、targetSdk、buildTools、Tools、Platform-tools的概念和使用,以及对项目全局配置的 com.android.tools.build:gradle:x.x.x 和 gradle版本的理解

<!-- more -->

# 前言 #

在开发中经常发现有AS有更新提示,在之前没有完全弄明白这些SDK,Tools的概念前都不敢轻易去更新,总担心更完就编译出错,API不能用等情况.所以对这些概念进行深度理解和应用建议.

# 参考文章 #

[如何选择 compileSdkVersion, minSdkVersion 和 targetSdkVersion](https://chinagdg.org/2016/01/picking-your-compilesdkversion-minsdkversion-targetsdkversion/ "如何选择 compileSdkVersion, minSdkVersion 和 targetSdkVersion")

[Android关于buildToolVersion与CompileSdkVersion的区别](https://blog.csdn.net/mooreliu/article/details/47167473)

[关于Android SDK里的compileSdk、minSdk、targetSdk、buildTools、Tools、Platform-tools](https://www.jianshu.com/p/ff3aa298e21b)

[Android中的Gradle插件说明](https://developer.android.com/studio/releases/gradle-plugin#revisions)

# compileSdk、minSdk、targetSdk到概念 #

## compileSdkVersion ##

compileSdkVersion 告诉 Gradle 用哪个 Android SDK 版本编译你的应用.使用任何新添加的 API 就需要使用对应 Level 的 Android SDK.

需要强调的是**修改 compileSdkVersion 不会改变运行时的行为**.当你修改了 compileSdkVersion 的时候，可能会出现新的编译警告、编译错误，但新的 compileSdkVersion 不会被包含到 APK 中：它纯粹只是在编译的时候使用。

因此我们强烈推荐**总是使用最新的 SDK 进行编译**,在现有代码上使用新的编译检查可以获得很多好处，避免新弃用的 API ，并且为使用新的 API 做好准备。

注意，如果使用 [Support Library](https://developer.android.com/topic/libraries/)(包括普通的 Support Library 和 Data Binding Library两个大类型) ，那么使用最新发布的 Support Library 就需要使用最新的 SDK 编译。例如，要使用 23.1.1 版本的 Support Library ，compileSdkVersion 就必需至少是 23 （大版本号要一致！）。通常，新版的 Support Library 随着新的系统版本而发布，它为系统新增加的 API 和新特性提供兼容性支持。

>综上, compileSdkVersion 这一项内容放心的使用最新版的即可,因为编译版本是向前兼容的,而且使用编译版本可让我们方便的知道最新版本的sdk中的api更新情况

## minSdkVersion ##

如果 compileSdkVersion 设置为可用的最新 API，那么 **minSdkVersion 则是应用可以运行的最低要求** minSdkVersion 是 Google Play 商店用来判断用户设备是否可以安装某个应用的标志之一。

在开发时 minSdkVersion 也起到一个重要角色：[lint](https://developer.android.com/studio/write/lint?utm_campaign=adp_series_sdkversion_010616) 默认会在项目中运行，它在你使用了高于 minSdkVersion 的 API 时会警告你，帮你避免调用不存在的 API 的运行时问题。如果只在较高版本的系统上才使用某些 API，通常使用[运行时检查系统版本](https://developer.android.com/training/basics/supporting-devices/platforms?utm_campaign=adp_series_sdkversion_010616)的方式解决。

有一点: 你所使用的库，如 Support Library 或 Google Play services，可能有他们自己的 minSdkVersion 。你的应用设置的 minSdkVersion 必需大于等于这些库的 minSdkVersion 。例如有三个库，它们的 minSdkVersion 分别是 4, 7 和 9 ，那么你的 minSdkVersion 必需至少是 9 才能使用它们。

>综上, minSdkVersion标记了最低可安装的手机系统版本,就实际情况来说,现在一般的最低版本都是19(本人项目中都是使用的这个版本,没有出现过问题,实际上可以根据实际情况酌情提升到21版本,即5.0以上,因为Android中从5.0开始更新了很多实用的东西,具体可以查看[Android5-7重要新特性](https://zhaozehui.cn/2018/05/21/Android5-7%E9%87%8D%E8%A6%81%E6%96%B0%E7%89%B9%E6%80%A7/))

## targetSdkVersion ##

三个版本号中最有趣的就是 targetSdkVersion 了。 **targetSdkVersion 是 Android 提供向前兼容的主要依据**，在应用的 targetSdkVersion 没有更新之前系统不会应用最新的行为变化。这允许你在适应新的行为变化之前就可以使用新的 API （因为你已经更新了 compileSdkVersion 不是吗？）。

例如，[Android 6.0 变化](https://developer.android.com/about/versions/marshmallow/android-6.0-changes?utm_campaign=adp_series_sdkversion_010616)文档中谈了 target 为 API 23 时会如何把你的应用转换到[运行时权限](https://link.juejin.im/?target=http://android-developers.blogspot.com/2015/08/building-better-apps-with-runtime.html?utm_campaign=adp_series_sdkversion_010616&amp;utm_source=medium&amp;utm_medium=blog)模型上，[Android 4.4 行为变化](https://developer.android.com/about/versions/android-4.4?utm_campaign=adp_series_sdkversion_010616)阐述了 target 为 API 19 及以上时使用 set() 和 setRepeating() 设置 alarm 会有怎样的行为变化。由于某些行为的变化对用户是非常明显的（弃用的 menu 按钮，运行时权限等），所以将 **target 更新为最新的 SDK 是所有应用都应该优先处理的事情**。但这不意味着你一定要使用所有新引入的功能，也不意味着你可以不做任何测试就盲目地更新 targetSdkVersion ，**请一定在更新 targetSdkVersion 之前做测试！**你的用户会感谢你的。

## buildTools、Tools、Platform-tools ##

buildTools、Tools、Platform-tools这3个东西其实都是开发工具，即它的版本更新并不会影响运行的APP，只是工具上的升级。

在 build.gradle 中的 buildToolsVersion 版本号一般是API-LEVEL.0.0，其中API-LEVEL要大于等于compileSdkVersion。

在前面的compileSdkVersion解释中建议选用最新的SDK Version，so，buildToolsVersion也建议选择最新的版本号。build.gradle中这2个的修改可以让你体验最新的API和工具。

至于Tools、Platform-tools这2个东西，直接更新最新吧。Only 工具。

> 实际上,在Android3.1.0版本之后,buildTools无需再指定,在 [Android中的Gradle插件说明](https://developer.android.com/studio/releases/gradle-plugin#revisions) 

![Android Studio Gradle Tools 3.1.0 版本及以上的配置](http://zhaozehui.cn/images/blogiamges/image_10.png "Android Studio Gradle Tools 3.1.0 版本及以上的配置")

# classpath 'com.android.tools.build:gradle:x.x.x'如何指定 #

![](http://zhaozehui.cn/images/blogiamges/image_11.png "Gradle更新提示")

经常在导入一个项目以后弹这个提示框,以前是不敢更新的.以后就可以放心大胆的更新了.

更新之后可能会出现Error:Could not find com.android.tools.build:gradle:x.x.x等问题, 这个问题经常出现在导入项目的时候无限building最后修改了 classpath 的情况 .所以这个应该怎么配置需要解释一下.

在 Android Studio 更新的时候,经常的Android的关于Gradle的插件也会更新,这个时候的更新配置是这样的,Android 是通过 Gradle来管理项目的,但是 Gradle不是就是为了Android而生的,所以Android中也是有插件来对Gradle进行依赖并管理的,所以在指定classpath 的时候实际上指定的是 Android 依赖Gradle的这个插件 , 这个插件的版本是在 AS 中的,其目录位于: Android Studio安装目录/gradle\m2repository\com\android\tools\build\gradle 中:
![](http://zhaozehui.cn/images/blogiamges/image_12.png "Gradle插件位置")

而gradle版本的位置可以在 AS 的 File - Settings - Build,Execution,Deployment - Gradle - Service directory path中
一般在 C://User/xxx/.gradle 文件夹下 

一般在安装 AS 的时候会携带最新的 Gradle插件 和Gradle 版本

可以参考一些问题例如: [Android Studio更新后出现Error:Could not find com.android.tools.build:gradle:3.4.1.](https://www.jianshu.com/p/a4bf2c5969e2 "classpath更新后可能出现找不到的情况")

# 总结 #

经过上面的深入了解后，总结以下：

当AS提示Gradle或者Android SDK更新后，大胆更新吧，先全部下载下来.

更新完后，直接将compileSdkVersion 修改为最新的版本号，放心的更改，该完后如果有废弃API编译器还会给你提示。

minSdkVersion 和 targetSdkVersion 要慎重修改。除非核心代码中会提高minSdkVersion的版本号，其他的建议运行时判断版本号。
targetSdkVersion的修改要注意代码是否适应更新后的版本号，要测试完全，最典型的例子就是23版本的运行时权限问题的处理。如果targetSdkVersion提升到了23，如果代码没有进行运行时权限判断会直接崩溃。

# 导入项目的正确姿势 #

上过上面的了解以后,导入项目的时候可以通过一些配置让无限building byebye.

- 首先如果下载的项目中含有 gradle文件夹, 可以删掉, 一般使用默认本地的gradle即可,因为本地的gradle肯定可以用,而且绝大多数情况都要比下载的项目更新,会自动依赖 .gradle 中
- 修改 compileSdkVersion 编译版本,可以修改成最新版的,如果不想下载的话可以修改成自己已经下载的最新版的,可以通过 SDK Manager的SDK Platforms来查看自己已经 install的版本
- 修改 classpath 的配置,可以通过查看Gradle插件的版本来指定
    >这个地方需要注意,要根据gradle版本来进行修改, 例如 依赖的插件版本是 3.0 之前的依赖是 complite ,而之后是 implementation 等细节

Gradle插件版本和Gradle的版本要求如下
![](http://zhaozehui.cn/images/blogiamges/image_13.png "Android中Gradle插件版本和Gradle版本关系")

另外可以通过[Android Plugin for Gradle Release Notes](https://developer.android.com/studio/releases/gradle-plugin)来查看Gradle依赖的具体版本情况















