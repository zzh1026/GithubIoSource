---
title: 一种粗暴快速的Android全屏幕适配方案
comments: false
date: 2018-05-17 17:24:08
tags: Android适配
categories: 转载学习资料

---

#### 出处: [http://www.jianshu.com/p/b6b9bd1fba4d](http://www.jianshu.com/p/b6b9bd1fba4d)
#### github: [https://github.com/Firedamp/Rudeness](https://github.com/Firedamp/Rudeness)

## 一、现状 ##

由于Android碎片化严重，屏幕适配一直是开发中较为头疼的问题。面对市面上五花八门的屏幕大小与分辨率，Android基于dp与res目录名称来适配的方案已无法满足一次编写全屏幕适配的需求，为了达到最优的视觉效果，开发过程中总是需要花费较多资源进行适配。也有开发者给出了一些自己的解决方案。

<!-- more -->

首先来分析一下一些常见的解决方案的现状：

### 1.官方适配方案   ###

dp。dp是Android开发中特有的一个单位。与px不同，dp是基于屏幕像素密度的一种单位。在密度低的屏幕上或许1dp=1px，但在密度高的屏幕上可能1dp=4px。编写布局xml时，如果一个控件的长宽都使用dp来指定，那么能确保该控件在各种大小与分辨率的屏幕下的绝对大小都大致相当。也就是说无论在pad下还是大小屏手机下，我们实际看到的该控件的大小是差不多的：
!["小屏幕和大屏幕显示同样dp宽高的场景"](https://upload-images.jianshu.io/upload_images/2692140-1b35754155b610d6?imageMogr2/auto-orient/)

资源目录名。上图可见虽然使用dp确保了控件在不同屏幕中的绝对大小一致。这样的好处在于，在大小相近的屏幕中，无论分辨率多大都不会对布局造成影响；但是当屏幕大小相差较大时，仅保证控件的绝对大小看起来就有些问题了。在res目录下可以给各资源目录都加上例如’-1920x1080’等后缀来适配不同的屏幕，具体规则可见官网文档。这样可以针对不同的屏幕提供不同的布局，甚至针对pad与手机提供两套完全不同的布局样式。但是通常情况下，设计师并不会对不同屏幕提供不同的设计图，他们的需求仅仅是不同屏幕下控件对屏幕的相对大小一致，所以dp并不能满足这一点，而对各种屏幕适配一遍又显得略为繁琐，并且修改也较为麻烦。

>dp解决了同一数值在 不同分辨率 中展示 相同尺寸大小 的问题（即屏幕像素密度匹配问题），但却没有解决设备 尺寸大小匹配 的问题。（即屏幕尺寸匹配问题）。
>>注意：屏幕宽度和像素密度没有任何关联关系。

通常我们需要的适配是这样的：

![](https://upload-images.jianshu.io/upload_images/3490737-953d5d55e6a0c042.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

[百分比布局支持库](https://github.com/JulienGenoud/android-percent-support-lib-sample)。没有使用过，但是deprecated in API level 26.0.0-beta1。
![](http://zhaozehui.cn/images/blogiamges/image_1.png)

ConstraintLayout。百分比支持库deprecated之后推荐使用的布局。

### 2.玩家适配方案 ###

广大玩家的适配目的很明确，目的就是要确保控件在不同屏幕的相对大小一致，看起来一毛一样的。以一位大神玩家的两种适配方案为例：

[AndroidAutoLayout](https://github.com/hongyangAndroid/AndroidAutoLayout) 和 [Android 屏幕适配方案](https://blog.csdn.net/lmj623565791/article/details/45460089)

首先编写脚本将长度转换成各分辨率下的长度，缺点是难以覆盖市面上的所有分辨率。
AutoLayout支持库。该库的想法非常好：对照设计图，使用px编写布局，不影响预览；绘制阶段将对应设计图的px数值计算转换为当前屏幕下适配的大小；为简化接入，inflate时自动将�各Layout转换为对应的AutoLayout，从而不需要在所有的xml中更改。但是同时该库也存在以下等问题：

- 扩展性较差。对于每一种ViewGroup都要对应编写对应的AutoLayout进行扩展，对于各View的每个需要适配的属性都要编写代码进行适配扩展；
- 在onMeasure阶段进行数值计算。消耗性能，并且这对于非LayoutParams中的属性存在较多不合理之处。比如在onMeasure时对TextView的textSize进行换算并setTextSize，那么玩家在代码中动态设置的textSize都会失效，因为在每次onMesasure时都会重新被AutoLayout重新设置覆盖。
- issue较多并且作者已不再维护。
- 还有一个比较实际的问题就是 , 当宽高写死在 Manifest中的时候, ui因为某种原因给的图换风格了(简单点就是换了一个ui可能给不一样宽高的图),这就很麻烦了.

## 二、想法 ##
对于大小差异较大的屏幕，本不该使用同一套设计方案，否则大屏的优势没有完全体现出来，从官方的适配方案也似乎是表达了这个意思。但是在实际设计与开发中，对于一个普通的App，很少有项目有意愿有精力来对各屏幕来分别设计与开发一套设计方案来适配。

通常的一个简单的适配需求是：假如设计图宽度为200，一个控件在设计图上标注的长度为3，那么该控件长度相当于总宽度的3/200，那么我们希望在任何大小的屏幕上该控件所表现的长度都为屏幕宽度的3/200。

个人觉得AutoLayout的设计思想非常优秀，但是将LayoutParams与属性作为切入口在mesure过程中进行转换计算的方案存在效率与扩展性等方面的问题。那么Android计算长度的收口在哪里，能不能在Android计算长度时进行换算呢？如果能在Android计算长度时进行换算，那么就不需要一系列多余的计算以及适配，一切问题就都迎刃而解了。

经过一番寻觅，发现系统进行长度计算的收口为TypedValue中的applyDimension函数，传入单位与value将其计算为对应的px数值。

        public static float applyDimension(int unit, float value, DisplayMetrics metrics) {
            switch (unit) {
            case COMPLEX_UNIT_PX:
                return value;
            case COMPLEX_UNIT_DIP:
                return value * metrics.density;
            case COMPLEX_UNIT_SP:
                return value * metrics.scaledDensity;
            case COMPLEX_UNIT_PT:
                return value * metrics.xdpi * (1.0f/72);
            case COMPLEX_UNIT_IN:
                return value * metrics.xdpi;
            case COMPLEX_UNIT_MM:
                return value * metrics.xdpi * (1.0f/25.4f);
            }
            return 0;
        }

- 可以看见换算方法非常简单，而DisplayMetrics的所有属性都是public的，不用反射就能修改；
- pt的原意是长度单位磅，根据当前屏幕与设计图尺寸将metrics.xdpi 进行修改就可以实现将pt这个单位重定义成我们所需要的相对长度单位，使修改之后计算出的1pt实际对应的px/屏幕宽度px=1px/设计图宽度px。
- 而这个DisplayMetrics从哪来？从源码中可以看出一般为mContext.getResources().getDisplayMetrics()，这个mContext即为所在Activity；
- 横竖屏切换等Configuration的变化会导致DisplayMetrics的重新计算还原；
- px,dp与sp都是平时常用的单位，而pt,in与mm几乎没有看见过，从这些不常见的单位下手正好可以不影响其他常用的单位。

基于以上几点，便有了以下方案。

## 三、方案 ##

本适配方案的目标是：完全按照设计图上标注的尺寸来编写页面，所编写的页面在所有大小与分辨率的屏幕上都表现一致，即控件在所有屏幕上相对于整个屏幕的相对大小都一致（看起来只是将设计图等比缩放至屏幕宽度大小）。

- 核心。使用冷门的pt作为长度单位，按照上述想法将其重定义为与屏幕大小相关的相对单位，不会对dp等常用单位的使用造成影响。
- 绘制。编写xml时�完全对照设计稿上的尺寸来编写，只不过单位换为pt。假如设计图宽度为200，一个控件在设计图上标注的长度为3，只需要在初始化时定义宽度为200，绘制该控件时长度写为3pt，那么在任何大小的屏幕上该控件所表现的长度都为屏幕宽度的3/200。如果需要在代码中动态转换成px的话，使用�

        TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_PT, value, metrics)。

- 预览。实时预览时绘制页面是很重要的一个环节。以1334x750的设计图为例，为了实现于正常绘制时一样的预览功能，创建一个长为1334磅，宽为750磅的设备作为预览，经换算约为21.5英寸((sqrt(1334^2+750^2))/72)。预览时选择这个设备即可。
- 代码处理。在activityonCreate时修改DisplayMetrics即可，推荐写在基类或ActivityLifecycleCallbacks中

        Point size = new Point();
        activity.getWindowManager().getDefaultDisplay().getSize(size);
        context.getResources().getDisplayMetrics().xdpi = size.x / designWidth * 72f;

这样绘制出来的页面就跟设计图几乎完全一样，无论大小屏上看起来就只是将设计图缩放之后的结果。

适配前（左图API19 400x800， 右图API24 1440x2560）：

![](https://upload-images.jianshu.io/upload_images/3490737-d5add2f4b91cc383.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

适配后（左图API19 400x800， 右图API24 1440x2560）：

![](https://upload-images.jianshu.io/upload_images/3490737-775011f0567ceb10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

代码及demo见[github](https://github.com/Firedamp/Rudeness)







