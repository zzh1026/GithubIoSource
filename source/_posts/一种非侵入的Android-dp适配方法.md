---
title: 一种非侵入的Android-dp适配方法
comments: false
date: 2018-06-04 16:47:22
tags: Android适配
categories:
    - Android
    - Android适配

---

# 简介 #

Android的屏幕适配文章已经是铺天盖地了，我之前也推荐过一种使用pt方式的适配 [一种粗暴快速的Android全屏幕适配方案](https://zhaozehui.cn/2018/05/17/%E4%B8%80%E7%A7%8D%E7%B2%97%E6%9A%B4%E5%BF%AB%E9%80%9F%E7%9A%84Android%E5%85%A8%E5%B1%8F%E5%B9%95%E9%80%82%E9%85%8D%E6%96%B9%E6%A1%88/ "一种粗暴快速的Android全屏幕适配方案")，这种方法我推荐的原因是：

- 使用pt作为单位，这个单位一般不常用，而且有收口。

但是这种方式也有很大的弊端

<!-- more -->

- Application的onCreate中需要引用

        //设计图标注的宽度
        int designWidth = 750;
        new RudenessScreenHelper(this, designWidth).activate();

当然还有其他的一些比如首次进入界面可能没有进行适配的问题，所以一直没敢用在项目中

而且现在网上基本都是使用px适配，即每种屏幕分辨率的设备需要定义一套dimens.xml文件。但是有些手机又虚拟按键，明确下面两个概念比较重要。

- 这是 状态栏：

![状态栏图片](https://upload-images.jianshu.io/upload_images/4776932-7d2d0ef473ae2060.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700 "状态栏图片")

- 这是导航栏：

![导航栏图片](https://upload-images.jianshu.io/upload_images/4776932-21527132218fca79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700 "导航栏图片")

在px适配的时候，其适配文件适配的是包含导航栏的高度的屏幕高，但是这个导航栏并非所有手机都有，所以就会出现一种情况，那就是当使用px来严格适配了一张充满整个屏幕的ui时候，那么因为导航栏的存在会使得实际屏幕的可用情况出现改变，这也是代码中获取屏幕宽高中的可用宽高和真实宽高可能不一致的原因。而且px适配的时候弊端也非常大：

- 每种屏幕分辨率都需要一套 dimens.xml文件来适配 ，apk包会变得很大，因为Android屏幕碎片化太严重使用，可以说这种方式有点不现实。
- 由于导航栏的问题，使得px适配可能出现适配错误的情况

后来出现的 [AndroidAutoLayout](https://github.com/hongyangAndroid/AndroidAutoLayout) 使用了动态计算屏幕的方式来进行适配，但是

- 需要在AndroidManifest中注明你的设计稿的尺寸，
- 让Activity继承自AutoLayoutActivity或者使用AutoLinearLayout等布局
- 已经不再维护

这样的代价有点大，首先 标明尺寸 意味着这样的库一旦使用起来，依赖会非常严重，其次使用自动化的布局导致扩展比较难。

**归根结底，在使用上述的方式进行适配，pt的方式有侵入性，AutoLayout方式严重侵入性，侵入性意味着我一旦使用这个库后期修改非常困难，因为实际开发中，可能ui设计师离职换了一个新的设计给了一套新的ui图，虽然可能性比较小但是这种侵入性很难让人接受。px的使用dimens文件方式侵入性小，但是一个是包变的太大，另一个是px使用起来比较麻烦**

**我需要一种侵入性很小，或者没有侵入性，同时不会对包有太大影响甚至没有影响，同时还得真的实现适配的结果。这样方便我以后修改或者扩展。**

# 推荐的一些文档 #

在适配方面，有一些文章可以更加清晰更加明确的说明适配的重要性，比如:

[Android屏幕适配全攻略(最权威的官方适配指导)](https://link.jianshu.com/?t=https%3A%2F%2Fblog.csdn.net%2Fzhaokaiqiang1992%2Farticle%2Fdetails%2F45419023)

## 从上述的文章中，摘抄出一些个人感觉比较重要的结论： ##

- 对于具有相同像素密度的设备来说，像素越高，尺寸就越大（这个点很重要，也很好理解）

## 一些比较重要的概念和理解 ##

### 屏幕尺寸（长度单位） ###

屏幕尺寸指屏幕的对角线的长度，单位是英寸，1英寸=2.54厘米

比如常见的屏幕尺寸有2.4、2.8、3.5、3.7、4.2、5.0、5.5、6.0等

因此也可以说一款手机宽多少寸，高多少寸，当一个手机只说屏幕是多少寸的时候，就是说手机屏幕的对角线长度

### 屏幕分辨率 ###

屏幕分辨率是指在横纵向上的像素点数，单位是px，1px=1个像素点。一般以纵向像素 * 横向像素，如1960*1080。

### 屏幕像素密度 ###

屏幕像素密度是指每英寸上的像素点数，单位是dpi，即“dot per inch”的缩写。屏幕像素密度与屏幕尺寸和屏幕分辨率有关，在单一变化条件下，屏幕尺寸越小、分辨率越高，像素密度越大，反之越小。

dpi代表的是每英寸上的像素点，代码中获取这个值得方式为：

    //获取手机的dpi值
    DisplayMetrics displayMetrics = new DisplayMetrics();
    getWindowManager().getDefaultDisplay().getMetrics(displayMetrics);
    float densityDpi = displayMetrics.densityDpi;

### dp、dip、dpi、sp、px ###

px我们应该是比较熟悉的，前面的分辨率就是用的像素为单位，大多数情况下，比如UI设计、Android原生API都会以px作为统一的计量单位，像是获取屏幕宽高等。

dip和dp是一个意思，都是Density Independent Pixels的缩写，即密度无关像素，上面我们说过，dpi是屏幕像素密度，假如一英寸里面有160个像素，这个屏幕的像素密度就是160dpi，那么在这种情况下，dp和px如何换算呢？在Android中，规定以160dpi为基准，1dip=1px，如果密度是320dpi，则1dip=2px，以此类推。

而sp，即scale-independent pixels，与dp类似，但是可以根据文字大小首选项进行放缩，是设置字体大小的御用单位。

> dip这个值： 在 160dpi中的 1dip = 1px ，这个是标准，是指定的标准，而非使用某种方式计算出来的标准，所以在这个标准的前提下就有如下公式： 

    1dp = 1dip = (任意一个手机的 densityDpi 即 dpi)/160 (px) = density

Android中获取该值的方式为：

    //获取手机的 1 dp的像素值
    float density = displayMetrics.density;

    即 1dp = 1dip = density px

所以： 一款手机的宽有多少dp可以用如下公式来计算

    //dp宽度为 手机屏幕宽/每个dp的px值
    float v = displayMetrics.widthPixels / density（density = （dpi/160））;

反过来说， 手机的宽度为：

    //手机的宽度 = 手机宽的dp值 * （手机像素密度 / 160的规定比例）
    float width = dp * (dpi/160)；

    手机的 dpi 就现在来说绝大多数 从xhdpi（240dpi）到xxxhdpi（640dpi）不等 ，即比例在 1.5 到 4不等
    手机的宽度来说绝大多数从 720px 到 1440px 不等，实际上，绝大多数的手机宽的dp值大致在 300dp 到 500dp这个区间内，这样的话其实就将适配的区间减小了。

### mdpi、hdpi、xdpi、xxdpi ###

名称|	像素密度范围
mdpi|	120dpi~160dpi
hdpi|	160dpi~240dpi
xhdpi|	240dpi~320dpi
xxhdpi|	320dpi~480dpi
xxxhdpi|	480dpi~640dpi

# px与dp适配的原理 #

## px适配原理 ##

根据设备屏幕的分辨率各自写一套dimens.xml文件，然后根据一个基准分辨率（例如720x1080），将宽度分成720份，取值为1px——720px，将高度分成1080份，取值为1px——1080px。生成各自dimens.xml文件对应的值。

## dp适配原理： ##

dp适配原理与px适配一样，区别就在于px适配是根据屏幕分辨率，即拿px值等比例缩放，而dp适配是拿dp值来等比缩放而已。

- 既然原理都一样，都需要多套dimens.xml文件，为什么说dp适配就比px适配好呢？
因为px适配是根据屏幕分辨率的，Android设备分辨率一大堆，而且还要考虑虚拟键盘。而dp适配无论手机屏幕的像素多少，密度比值多少，80%的手机的最小宽度dp值(widthPixels / density)都为360dp，这样就大大减少了dimens.xml文件。

- px适配会根据设备的分辨率去找对应的dimens.xml文件（如下图，运行在分辨率为1920x1080的手机上，系统会自动找到对应的values-1920x1080文件），那dp适配呢？

![](https://upload-images.jianshu.io/upload_images/5382223-4f42a5637f53f582.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/259 "使用px适配")

dp适配也是一样的，只不过dp适配是根据“最小宽度（Smallest-width）限定符”来找的，即如果当前设备最小宽度（以 dp 为单位）为400dp，那么系统会自动找到对应的values-sw400dp文件夹下的dimens.xml文件，如图

![](https://upload-images.jianshu.io/upload_images/5382223-c59b8353245a0ebd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/207 "使用dp适配")

# 关于最小宽度的资源限定符的说明 #

可以查看 [Android 适配时资源限定符的说明](https://blog.csdn.net/shishuinianshang/article/details/76154913)

> 在屏幕 尺寸相差不大的情况下，dp可以使不同分辨率的设备上展示效果相似。但是在屏幕尺寸相差比较大的情况下，dp就失去了这种效果。所以需要以下的限定符来约束，采用多套布局，数值等方式来适配。

android3.2之后引入的，目前推荐使用的适配方式

![](http://zhaozehui.cn/images/blogiamges/image_15.png "最小宽度限定符的使用方式")

强调一点： 

设备宽度的dp计算方法：

**dp = 屏幕像素宽度/(屏幕像素密度/160)   160是基准屏幕像素密度    这个用来计算以上的sw后面的数值**

通用公式：

dp = px/(dpi/160)

px = dp*(dpi/160)

# 适配姿势 #

使用Android Studio的插件：[android阿杜](https://blog.csdn.net/fesdgasdgasdg)的[https://github.com/mengzhinan/ScreenMatch](https://github.com/mengzhinan/ScreenMatch)来生成这些文件


## 工具使用步骤： ##

### 在Android Studio中安装ScreenMatch插件，如图： ###
![](https://upload-images.jianshu.io/upload_images/5382223-89529b5f508ae64a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 在项目的默认values文件夹中需要一份dimens.xml文件 ###

![](https://upload-images.jianshu.io/upload_images/5382223-021e947fcff9a39b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/408)

### 执行生成 ###

插件安装好后，在项目的任意目录或文件上右键，选择ScreenMatch选项。如图：

![](https://upload-images.jianshu.io/upload_images/5382223-4653a6677ba1c3b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/465)

### 然后选择在哪个module下执行适配。点击确定就会执行生成命令 ###

### 然后再看看res目录下会自动生成一堆dimens.xml文件，如下： ###
![](http://zhaozehui.cn/images/blogiamges/image_14.png "使用最小宽度限制符")

通过上面的步骤就已经生成了所有设备对应的dimens.xml文件。

因为默认生成的是下列最小宽度dp的dimens.xml文件
384,392,400,410,411,480,533,592,600,640,662,720,768,800,811,820,960,961,1024,1280,1365，如果不需要或者需要增加某些dp值的dimens.xml文件，则需要修改配置文件，即screenMatch.properties文件（修改前先删除之前生成的全部dimens.xml文件）。配置文件在我们执行完成上面的命令后，会在项目的目录下自动生成，如下：

![](https://upload-images.jianshu.io/upload_images/5382223-abd43dec273ae097.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/472)

打开文件，修改下面的值即可。如下只需要适配384,392,400,410,411的值，不需要适配480,533,592,600,640,662,720,768,800,811,820,960,961,1024,1280,1365的值

![](https://upload-images.jianshu.io/upload_images/5382223-7afd8a8a1ae35287.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

其中base_dp=360代表widthDP基准值，一般都是360dp，不建议更改，除非你对屏幕适配原理有深刻的见解。

**我自己的使用方式是新建一个项目，然后使用工具生成我说需要的所有dp值，然后将生成的结果放在res目录下**

## 根据设计图标注，在布局写上对应的值。 ##

在安卓中，系统密度为160dpi的中密度手机屏幕为基准屏幕，即320×480的手机屏幕。在这个屏幕中，1dp=1px。320x480分辨率对应的其他分辨率的比例如下：

![](https://upload-images.jianshu.io/upload_images/5382223-afb77e1f091db0bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

所以，如果UI给的是720x1280分辨率的图， 那么dp = px / 2， 给的是1080x1920分辨率的图，那么 dp = px / 3，即根据比例即可。

举例：UI在720x1280上做的图，其中一个按钮的宽高分辨为：宽720px，高为100px，字体大小为30px，在布局中则这样使用：

    <Button
        android:layout_width="@dimen/dp_360"
        android:layout_height="@dimen/dp_50"
        android:textSize="@dimen/sp_15"/>

> 注： 偶尔会有ios的设计图是 750*1334 的图，这个的比例为 2.08 ，即如果是 720的图 /2 即可，但是 ios的750需要 /2.08；

# 参考 #

使用步骤截取的 [推荐一种非常好用的Android屏幕适配](https://www.jianshu.com/p/1302ad5a4b04)

使用原理和对比参考 [Android屏幕适配dp、px两套解决办法](https://blog.csdn.net/fesdgasdgasdg/article/details/52325590)

# 适配优势 #

- 侵入性方面来说，这套适配方案没有侵入性
- 适配绝大多数手机增加大约200k左右，完全能接受（本人实际项目中适配的从 320 到 600）

缺点：

- 适配了宽，不适配高。

上述缺点其实实际开发中基本没有什么影响，因为如果宽适配了以后ui要的撑满高的需求并不多。一般使用的是 NestScrollView来包裹，高在必须的情况下可以使用 ConstraintLayout 来对控件进行宽高比的设置。
    







