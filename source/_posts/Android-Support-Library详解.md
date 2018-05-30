---
title: Android Support Library详解
comments: false
date: 2018-05-30 14:23:13
tags: Support-v4 ,Support-v7 ,Android Support Library
categories: Support Library

---

# 简介 #

以前在Android开发中,经常会出现乱七八糟的包冲突,十次里面至少八次是因为v4包冲突,为啥v4包这么爱冲突?Android官方为什么要提供支持包?都提供哪些支持包?这些支持包又提供了什么特性?开发者又应该如何选择使用这些支持包？......

<!-- more -->


# Android Support V4, V7, V13是什么？ #

本质上就是三个java library。

# Android官方为什么要提供支持包？ #

为什么官方向开发者在提供了android sdk之外，还要提供一些零碎的开发支持jar包,因为如果在低版本Android平台上开发一个应用程序，而应用程序又想使用高版本才拥有的功能，就需要使用Support库。

下图罗列了已经发布的Android版本及其对应的开发sdk的级别。
![](https://upload-images.jianshu.io/upload_images/1621638-ab90ba556333d9a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

至于为什么提供支持包官方给出了大致三个原因：

- 向后兼容
 
    如，我们开的App需要支持的minSdkVersion=9，targetSdkVersion=11，在程序里使用了android 3.0 (API level 11)提供的ActionBar类，使用compileSdkVersion=11成功编译出apk。在android 3.0的设备上完美运行，但是app在android 2.3的设备就会crash，报找不到ActionBar的错误。这很好理解，因为旧版本上没有新版本里新增的类。为了避免使用了最新功能开发的app只能在最新系统的设备上运行的尴尬，官方把新版系统framework中新增加的接口提出来放到了Android Support Library（支持包）中，开发者在遇到上面的情况时，就可以使用支持包中具有同样功能的ActionBar类，这个支持包会打包进App里，这样使用了新版本系统上功能的App也可以向后兼容以前的老系统版本设备了。
    使用支持包中的类除了让我们免于判断App运行的系统版本外，还可以使App在各个版本保持同样的用户体验。如在5.0以下系统使用material design。

>App编译时用的android sdk（android.jar）不会打包进我们的App中。因为App编码是使用android.jar中的接口就是android设备里系统框架层（framework）对外提供的接口。

- 提供不适合打包进framework的功能

    Android官方对App开发提供了推荐设计，希望Android应用都有相对一致的交互设计来减少用户的使用成本，希望三方App类似系统应用从而完美融入到Android生态系统中。但是这都仅仅是推荐，不要求开发者一定要这样，如果有这种需求就可以使用官方支持包提供的这些功能，避免重复造轮子。如支持包中的DrawerLayout、Snackbar等类都是这种情况。

- 为了支持不同形态的设备

    通过使用支持包来在不同形态设备上提供功能，如手机、电视、可穿戴设备等。


# 官方提供了哪些支持包，都有哪些特性？ #

现在Android官方发布了下面13类支持库，不同的支持库包含不同特征，适用的Android版本也不相同。通常情况下我们都使用到v4和v7这两个集合库，因为这两个库支持的android系统版本范围比较广，官方推荐的UI设计样式相关类也都在这两集合库中。

![](https://upload-images.jianshu.io/upload_images/1621638-1f66aafb225df824.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700 "Android Support Library")

## v4 Support Libraries ##

- v4库被设计在Android 2.3 (API level 9)及其以上系统中使用。 Support Library的第1版（２０１１年３月发布）就只包含v4库，当时v4库只是一个库，支持Android 1.6 (API level 4)及其以上版本，这也是v4名字的由来。
- 随着系统的迭代现在Android 1.6设备已经很少了，官方在Support Library的第24.2.0版本（２０１6年8月发布）的时候移除了对Android 2.2 (API level 8)及其以下版本的支持，但是名字依然是v4。
- v4悠久的历史长期的发展造就了它较大的体积。也是在24.2.0这个版本Goggle将原来的单个v4库拆分成了5个子库，我们在使用的时候可以直接依赖某个子库，从而减少依赖包的大小。

**上面有一点需要注意: 24.2.0这个版本Goggle将原来的单个v4库拆分成了5个子库,即 在 24.2.0 之前依赖 v4 包的时候,v4包的存在形式是 v4包的一个总的 jar 包, 之后的版本在依赖 v4 包的时候出现了5个包,下面三张图分别是 v24.2.0之前 , 之后 ,和最新的(27.1.1)版本的 v4 包的变化**

![](http://zhaozehui.cn/images/blogiamges/image_9_v23.png "24.2.0版本之前的v4依赖方式")

![](http://zhaozehui.cn/images/blogiamges/image_9_v24.png "24.2.0版本的v4依赖方式")

![](http://zhaozehui.cn/images/blogiamges/image_9_v27.png "27.1.1最新版的v4依赖方式")

可以看到 , 最新版的v4去除了 media 的支持包, 同时 v4 的空包也不在了,v4 包也是在不断发展和变化的

现在 ,Gradle编译脚本中整个v4库的依赖语句如下：

    implementation 'com.android.support:support-v4:24.2.1'  

>gradle中jar依赖语句格式如 compile 'jar文件组（group/命名空间）:jar文件名（name）:jar文件版本（version）'。所以上面的语句意思是依赖Android支持库中第24.2.1版的support-v4库。由于在24.2.0版本support-v4库已经被拆分成5个子库，所以如下图所示依赖24.2.1版本的support-v4库除了导入support-v4库外还会导入它的5个子库，这个版本的support-v4库本身是一个空的包，所有具体的实现都在它依赖的子库中。下面依次看下v4库拆分出来的5个子库。

![](https://upload-images.jianshu.io/upload_images/1621638-e50b38383771e40c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700 "24.2.1版本的v4包结构")

### v4 compat library ###

为一些框架的API提供兼容性包装。如，Context.obtainDrawable()、View.performAccessibilityAction()等。
Gradle编译脚本中v4 compat库的依赖语句：

    implementation 'com.android.support:support-compat:24.2.1'

### v4 core-utils library ###

提供了一些工具类。如，AsyncTaskLoader、PermissionChecker等。
Gradle编译脚本中v4 core-utils库的依赖语句：

    implementation 'com.android.support:support-core-utils:24.2.1'

### v4 core-ui library ###

提供很多UI相关组件。如，ViewPager、NestedScrollView、ExploreByTouchHelper等。
Gradle编译脚本中v4 core-ui库的依赖语句：

    implementation 'com.android.support:support-core-ui:24.2.1'

### v4 media-compat library ###

多媒体框架相关部分。如，MediaBrowser、MediaSession等。
Gradle编译脚本中v4 media-compat库的依赖语句：

    implementation 'com.android.support:support-media-compat:24.2.1'

### v4 fragment library ###

跟fragment相关部分。v4这个子库依赖了其他4个子库，所以我们一旦依赖这个库就会自动导入其他4个子库，这跟直接依赖整个support-v4效果类似。Gradle编译脚本中v4 fragment 库的依赖语句如下：

        implementation 'com.android.support:support-fragment:24.2.1'

## v7 Support Libraries ##

注意这里的Library用的也是复数，说明v7库和v4一样也是很多库的集合，不同的是v7各个库不是后来拆分出来的，而是最初发布时就是以各个独立的库的形式发布的，如发布的最早的v7库v7 gridlayout library。这些库的共同之处是发布之初都是支持Android 2.1 (API level 7)及其以上版本，所以把他们统称为v7支持库。需要注意的24.2.0版本以后的v7支持库支持范围也是Android 2.3 (API level 9)及其以上版本了，这是因为v7依赖的v4库支持版本范围改变了，这点在v4支持库小节有介绍。

**这里要注意的一点是:v7包v4包,也就是说如果项目里面继承了v7包,则默认也将v4包集成进去了,同时v4包和v7包的版本是一致的,所以在实际项目中,直接继承v7的appcompat 包即可间接的依赖v4包,所以正常情况下只需要依赖v7包就可以了,v4包一般不需要特地去额外添加依赖.**

v7库集合里有7个子库，使用时根据需要选择导入哪些库。

### v7 appcompat library ###

支持UI设计样式、 material design相关，如ActionBar、AppCompatActivity、Theme等。

Gradle编译脚本中v7 appcompat库的依赖语句：

    implementation 'com.android.support:appcompat-v7:27.1.1'

>依赖 v7-appcompat包会间接依赖 v4包

### v7 cardview library ###

支持cardview控件，使用material design语言设计，卡片式的信息展示，在电视App中有广泛的使用。

Gradle编译脚本中v7 cardview库的依赖语句：

    implementation 'com.android.support:cardview-v7:27.1.1'

### v7 gridlayout library ###


支持gridlayout布局(网格布局)。gridlayout可以减少布局嵌套层次从而提高性能,但是学习难度比较大. 用来 [制作计算器](https://www.jianshu.com/p/916aa3825fff "Android六大布局之GridLayout讲解、用GridLayout制作计算器界面。")是一个很方便的布局

Gradle编译脚本中v7 gridlayout库的依赖语句：

    implementation 'com.android.support:gridlayout-v7:27.1.1'

### v7 mediarouter library ###

该库提供了 MediaRouter、MediaRouteProvider等与Google Cast相关的类。

Gradle编译脚本中v7 mediarouter库的依赖语句：

    implementation 'com.android.support:mediarouter-v7:27.1.1'

### v7 palette library ###

该库提供了palette类，使用这个类可以很方便提取出图片中主题色。比如在音乐App中，从音乐专辑封面图片中提取出专辑封面图片的主题色，然后将播放界面的背景色设置为封面的主题色，随着播放音乐的改变，播放界面的背景色也会巧妙的跟着改变，从而提供更好的用户体验。

Gradle编译脚本中v7 palette库的依赖语句：

    implementation 'com.android.support:palette-v7:27.1.1' 

### v7 recyclerview library ###    

该库提供了recyclerview类。这个库必备.

Gradle编译脚本中v7 recyclerview库的依赖语句：

    implementation 'com.android.support:recyclerview-v7:27.1.1'

### v7 Preference Support library ###

这个库在设置界面常用到。提供了 CheckBoxPreference、ListPreference等类。

Gradle编译脚本中v7 preference support库的依赖语句：

    implementation 'com.android.support:preference-v7:27.1.1'

>这个库没用过,一般用不到

## v8 Support Library ##

v8支持库支持范围也是Android 2.3 (API level 9)及其以上版本。v8支持库集合中现在只有一个库。这个库支持渲染脚本计算框架。(实际项目中一般用不到)

## v13 Support Library ##

Android Support v13:这个包的设计是为了android 3.2及更高版本的，一般我们都不常用，平板开发中能用到。这个库跟v4 fragment library功能基本一样，也是提供兼容fragment相关内容。区别是v4 fragment library需要依赖v4支持库集合里的其它4个子库，而v13 support library依赖的是Android 3.2 (API level 13)及其以上版本framework。(实际项目中没专门用过这个库)

## v14 Preference Support Library ##

功能类似v7 Preference Support library，支持Android系统版本不一致，新增部分相关接口。

## v17 Preference Support Library for TV ##

功能类似v7 Preference Support library，支持Android系统版本不一致，新增部分相关接口，为电视设备App提供相应的UI。

## v17 Leanback Library ##

在电视设备上使用的库，主要是和YouTube相关的。

## Annotations Support Library ##

提供注解相关功能。

Gradle编译脚本中Annotations Support库的依赖语句：

    implementation 'com.android.support:support-annotations:27.1.1'  

## Design Support Library ##

这个库现在使用的也比较多，它提供了material design设计风格的控件。如，navigation drawers、floating action buttons (FAB)、snackbars、tabs等。

Gradle编译脚本中Design Support库的依赖语句：

    implementation 'com.android.support:design:27.1.1'  

## Multidex Support Library ##

Android的单个.dex文件最多能引用65536个方法，在这之后的方法就无法引用了。当我们的方法数超过这个限制后就需要分成多个dex文件，该库就是用来支持多个dex文件构建应用程序的。

Gradle编译脚本中Multidex Support库的依赖语句：

    implementation 'com.android.support:multidex:1.0.1'  

## Custom Tabs Support Library ##

提供了一种新的打开网页的方式。用不着.

## App Recommendation Support Library for TV ##

电视设备上用来提供视频内容推荐的。(没用过)

## Percent Support Library ##

google提供的百分比框架, 但是 deprecated in API level 26.0.0-beta1 .

![](http://zhaozehui.cn/images/blogiamges/image_1.png "google百分比支持库")

## Constraint Layout ##

google推荐的用来代替百分比框架的布局,可以进行百分比适配,约束布局相对来说好用很多

Gradle编译脚本中Constraint Layout库的依赖语句：

    implementation 'com.android.support.constraint:constraint-layout:1.1.0'

> constraint-layout有局限性,实际开发中没有想象中的好用,希望以后能扩展

# 如何选择使用支持包？ #

其实在了解了支持包特性之后，这个问题也就迎刃而解了，这里再做下总结。在使用Android Support Library之前我们需要通过sdk manager安装Android Support Repository，然后再在gradle编译脚本中添加如下依赖语句就可以了。

前面文章说过gradle中jar依赖语句格式如 compile jar文件组（group/命名空间）:jar文件名（name）:jar文件版本（version）。对于Android Support Library库的依赖语句jar文件名和jar文件版本两部分需要选择确定。

jar文件名：在选择之前要明确两件事，需要使用支持包的哪种特性、需要兼容的最低Android版本，然后就可以确定具体依赖哪个支持库。
jar文件版本：支持库的版本需要跟compileSdkVersion保持一致。

>注意：由于依赖的支持库会打包进apk，所以官方推荐开发者在编译时使用ProGuard工具预处理release版本的apk。ProGuard工具除了混淆源代码外，还会移除那些依赖的支持库中没有使用到的类，达到apk瘦身的效果。

# 参考 #

[Android Support Library的前世今生](https://www.jianshu.com/p/f5f9a4fd22e8 "Android Support Library的前世今生")

[https://www.jianshu.com/p/dfc55afe0f47](https://www.jianshu.com/p/dfc55afe0f47 "Android Support Library v4 模块拆分")

[https://www.jianshu.com/p/916aa3825fff](https://www.jianshu.com/p/916aa3825fff "GridLayout讲解、用GridLayout制作计算器界面")