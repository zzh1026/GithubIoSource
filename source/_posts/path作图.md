---
title: path作图
comments: false
date: 2018-05-18 13:41:37
tags: pathData,vector,VectorDrawable
categories: 
     - Android
     - Android View

---

## 简介 ##

Android Support Library 23.2 出来以后，在Android 5.0(API级别21)以前的系统中，也可以定义矢量drawables，即VectorDrawable。它可以在不失清晰度的情况下进行缩放。你仅仅需要需要一个矢量图片的资源文件，而不再需要为每个屏幕密度设置一个资源文件，在一定程度上可以减小项目的体积。

vector 标签下的最主要就是 pathData，其实pathData跟Android中 Path api对路径的定义规则是差不多的，当你掌握了 pathData 的语法，同时有一颗不轻易放弃的心，就可以通过一些简洁的指令完成几乎所有的图案。

<!-- more -->

本文参考了: [Android vector 标签 pathData 详解](https://blog.csdn.net/scott2017/article/details/51770620) 

## 效果 ##

![击掌](http://upload-images.jianshu.io/upload_images/1175492-725be47c52e14e61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "give me five.png")

    <!--上面的手掌对应的代码实现-->
    <vector xmlns:android="http://schemas.android.com/apk/res/android"
            android:width="24dp"
            android:height="24dp"
            android:viewportWidth="24"
            android:viewportHeight="24">
        <path
            android:fillColor="#000000"
            android:pathData="
                M22,23 q0,4 -4,4 h-7 q-2,0 -3,-1 T1,16 q-0.6,-0.8 0,-2 t5,3
                q1,1 2,0 T8,4 q0,-1 0.9,-1.1 t1.1,1 1.5,9 q0.25,0.5 0.5,0.5
                t0.5,-0.5 0,-11 q0.2,-1 1.1,-1.1 t1.1,1.1 1,11 q0.25,0.5 0.5,0.5
                t0.5,-0.5 0.5,-9 q0.2,-1 1,-1 t1,1 0.5,9 q0.25,0.5 0.5,0.5
                t0.5,-0.5 1.2,-6.5 q0.3,-1 1,-1 t0.8,1 -0.8,6 T22,23"/>
    </vector>


代码中引用方式

![](http://zhaozehui.cn/images/blogiamges/image_5.png)

## 基本的语法 ##

    M：move to 移动绘制点，作用相当于把画笔落在哪一点。
	L：line to 直线，就是一条直线，注意，只是直线，直线是没有宽度的，所以你什么也看不到。
	Z：close 闭合，嗯，就是把图封闭起来。
	C：cubic bezier 三次贝塞尔曲线
	Q：quatratic bezier 二次贝塞尔曲线
	A：ellipse 圆弧

    m：move to 移动绘制点，作用相当于在当前基础上把画笔落在哪一点。
	l：line to 直线，在当前基础上的一条直线，注意，只是直线，直线是没有宽度的，所以你什么也看不到。
	c：cubic bezier 在当前基础上的三次贝塞尔曲线
	q：quatratic bezier 在当前基础上的二次贝塞尔曲线
	a：ellipse 在当前基础上的圆弧

>有大小之分, 大写表示绝对值,小写表示相对值
>每个命令都有大小写形式，大写代表后面的参数是绝对坐标，小写表示相对坐标，相对于上一个点的位置。参数之间用空格或逗号隔开。

## 命令解释： ##

- M (x y) 把画笔移动到x,y，要准备在这个地方画图了。
- L (x y) 直线连到x,y，还有简化命令H(x) 水平连接、V(y)垂直连接。
- Z，没有参数，连接起点和终点
- C(x1 y1 x2 y2 x y)，控制点（x1,y1）（ x2,y2），终点x,y 。	(三阶贝塞尔曲线)
    - C x1,y1 x2,y2 x,y ( c dx1,dy1 dx2,dy2 dx,dy)
    - S x1,y1 x,y ( s dx1,dy1 dx, dy)
    - S指令跟T指令类似。S指令相对于C指令少了一个控制点，这个控制点就是上一次最后一个控制点相对上次的终点的中心对称点。
- Q(x1 y1 x y)，控制点（x1,y1），终点x,y (二阶贝塞尔曲线)
    - Q x1,y1 x,y ( q dx1,dy1 dx,dy)
    - T x,y ( t dx, dy)
    - T指令是在你画完一条贝塞尔曲线后，只需用T指令指定终点，就能画出一条平滑的贝塞尔曲线。控制点被默认为上一次的控制点关于上次终点的中心对称点。比如上次的控制点P1是(6,6)，终点P2是(8,10)， 那么使用T指令后默认控制点P1`为(10,14)。
- A (rx,ry x-axis-rotation large-arc-flag sweepflag x,y )
    - android:pathData=" M50,50 a10,10 1 1 0 1,0"
    - rx ry 椭圆半径
    - x-axis-rotation x轴旋转角度
    - large-arc-flag 为0时表示取小弧度，1时取大弧度（舍取的时候，是要长的还是短的）
    - sweep-flag 0取逆时针方向，1取顺时针方向 
    - x,y (dx,dy) 终点的位置(这个位置 是和 位置相关的, 要根据使用的是 A 还是 a 来确定)
    
## 命令详细规则 ##

pathData 的指令基本都是由字母跟若干数字组成，数字之间可以用空格或者逗号隔开 (其实逗号会被忽略掉，加上逗号只是一些习惯的问题)。一般来说指令字母分为大小写两种,大写的字母是基于原点的坐标系(偏移量)，即绝对位置；小写字母是基于当前点坐标系(偏移量)，即相对位置。

基于 Android屏幕, 取向右向下为正, 向左向上为负.

#### 移动 ####
- M x,y (m dx, dy) 移动虚拟画笔到对应的点，但是并不绘制。一开始的时候默认是在(0,0)。

#### 直线 #### 
- L x,y (l dx, dy) 从当前点划一条直线到对应的点。
- H x (h dx) 从当前点绘制水平线，相当于l x,0
- V y (v dy) 从当前点绘制垂直线，相当于l 0,y

        <vector xmlns:android="http://schemas.android.com/apk/res/android"
              android:width="24dp"
              android:height="24dp"
              android:viewportWidth="24"
              android:viewportHeight="24">
    
              <path
                  android:fillColor="#0000"
                  android:strokeColor="#000"
                  android:strokeWidth="0.2"
                  android:pathData=" M10,10 L10,15 L15,15 L10,10"/>
        </vector>

{% cq %} ![](http://upload-images.jianshu.io/upload_images/1175492-043c1978d8aca0ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) {% endcq %}

将上述代码 android:pathData=" M10,10 L10,15 L15,15 L10,10" 替换成以下代码效果相同

     android:pathData="M10,10 l 0,5 l 5,0 l-5,-5"  
     android:pathData="M10,10 V 15 H 15 L10,10"  
     android:pathData="M10,10 v 5 h 5 l-5,-5"

#### 闭合 ####

- Z(或z) 从结束点绘制一条直线到开始点，闭合路径

上面的图形型也可以由以下代码绘制

    android:pathData="M10,10 v 5 h 5 z"

#### 弧线 ####

- A rx,ry x-axis-rotation large-arc-flag,sweepflag x,y
- a rx,ry x-axis-rotation large-arc-flag,sweepflag dx,dy

        rx ry 椭圆半径
        x-axis-rotation x轴旋转角度
        large-arc-flag 为0时表示取小弧度，1时取大弧度（要长的还是短的）
        sweep-flag 0取逆时针方向，1取顺时针方向
        x,y (dx,dy) 终点的位置

这个弧线的指令比起直线就相对复杂得多了，7个参数容易搞混了。来看个例子

    android:pathData="M8,10 a4,6 0 1,1 6 6"

![](http://upload-images.jianshu.io/upload_images/1175492-c3e722abe817f717.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
红色(8,10) 是起点， X轴旋转的角度为0，取大弧，顺时针，蓝色(14,16) 是终点。

- 只将x-axis-rotation 改为30跟-30分别对应下面左边跟右边的效果，可见正数是按顺时针旋转负数按逆时针旋转。

![](http://upload-images.jianshu.io/upload_images/1175492-8f6c17cfffecb5f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

x-axis-rotation为30（左）、-30（右）

- 只将large-arc-flag 改为 0 android:pathData="M8,10 a4,6 0 0,1 6 6" 对应下图效果

![](http://upload-images.jianshu.io/upload_images/1175492-ce86320fe7a69d1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

large-arc-flag为 0

- 只将sweep-flag改为 0 android:pathData="M8,10 a4,6 0 1,0 6 6" 对应下图效果

![](http://upload-images.jianshu.io/upload_images/1175492-ed77fd5ec8b949c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sweep-flag为0

网上看到一张图，基本总结了弧线

![](http://upload-images.jianshu.io/upload_images/1175492-c79b5e4bf39363c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 二阶贝塞尔曲线 ####

- Q x1,y1 x,y ( q dx1,dy1 dx,dy)
- T x,y ( t dx, dy)

![](http://upload-images.jianshu.io/upload_images/1175492-67b35c33f0aeda7f.gif?imageMogr2/auto-orient/strip)

对于Q指令来说(x1,y1)就是P1这个控制点，(x,y)就是终点P2。P0当然就是执行指令前最后的位置。这里有一点需要说明的， q dx1,dy1 dx,dy 这个指令，两个控制点都是相对P0点的。

而T指令是在你画完一条贝塞尔曲线后，只需用T指令指定终点，就能画出一条平滑的贝塞尔曲线。控制点被默认为上一次的控制点关于上次终点的中心对称点。比如上次的控制点P1是(6,6)，终点P2是(8,10)， 那么使用T指令后默认控制点P1`为(10,14)。


举个例子：

     android:pathData="M4,10 Q6,6 8,10 Q10,14 12,10"  
     android:pathData="M4,10 Q6,6 8,10 T12,10" 
     android:pathData="M4,10 q2,-4 4,0 q2,4 4,0"  
     android:pathData="M4,10  q2,-4 4,0 t4,0"

这四个对应的都是下面的曲线

![](http://upload-images.jianshu.io/upload_images/1175492-b2c1fcf005c6463d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 三阶贝塞尔曲线 ####

- C x1,y1 x2,y2 x,y ( c dx1,dy1 dx2,dy2 dx,dy)
- S x1,y1 x,y ( s dx1,dy1 dx, dy)

![](http://upload-images.jianshu.io/upload_images/1175492-6b58a1da6d923644?imageMogr2/auto-orient/strip)

跟二阶类似，但是三阶有两个控制点，分别是P1(x1,y1)，P2(x2,y2)还有终点P3(x,y)。类似的， c dx1,dy1 dx2,dy2 dx,dy 所有的点都是相对于P0，即上一次的终点。

S指令跟T指令类似。S指令相对于C指令少了一个控制点，这个控制点就是上一次最后一个控制点相对上次的终点的中心对称点。还是同样举个例子：

     android:pathData="M4,10 C6,6 8,14 10,10 C12,6 14,14 16,10"
     android:pathData="M4,10 C6,6 8,14 10,10 S14,14 16,10"
     android:pathData="M4,10 c2,-4 4,4 6,0 c2,-4 4,4 6,0"
     android:pathData="M4,10 c2,-4 4,4 6,0 s4,4 6,0"

以上四个均会出现下图的效果

![](http://upload-images.jianshu.io/upload_images/1175492-5871e206b7b0338d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 实践 ##

终于把所有的指令搞清楚啦，那么接下来就来画一个比较简单的图案吧

![](http://upload-images.jianshu.io/upload_images/1175492-bdcb5a670356000a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先分析一下这个图的构成。它是一个圆角正方形同时里面有一个镂空的心型。先来看看正方形

    <vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="24dp"
        android:height="24dp"
        android:viewportHeight="40.0"
        android:viewportWidth="40.0">
    
        <!-- 将 vector的范围标识出来, 没有其他作用,可以删掉 -->
        <path
            android:pathData="M0,0 v40 h40 v-40 z"
            android:strokeColor="#fff"
            android:strokeWidth="0.2"></path>
    
        <path
            android:fillColor="#f24e4e"
            android:pathData="M8,4
                h24 q4,0 4,4 v24 q0,4 -4,4
                h-24 q-4,0 -4,-4 v-24 q0,-4 4,-4" />
    </vector>

上面viewportWidth为40，则把这张图分成40个单位。

在pathData中，首先移动到A(8,4)，然后水平移动了24个单位到B(32,4)。接着就是一条二阶贝塞尔曲线，起点是B，控制点是C(36,4)，终点是D(36,8)。然后再画一条垂直线到(36,32)…下面的操作都是差不多，就不多说的。这里注意一点，画这个正方形是从A-B-C-D，也就是顺时针的方向，这点对接下来画心型很重要。

画完了圆角正方形，接下来就要画一个镂空的心型。有同学说再加上一个path标签，设置不同颜色就可以。这方法乍一看挺不错，但是他实现的并不是镂空的效果，而是叠加的效果。镂空的话中间的心型是透明的，但是叠加就没办法让中间的心呈透明。

那怎么实现镂空的效果呢？其实关键就是前面说的方向。前面的正方形是顺时针的方向，如果我们在pathData 后面加上另一逆时针方向的路径，就会出现取反的效果，比如

    <path
        android:fillColor="#f24e4e"
        android:pathData="M8,4
            h24 q4,0 4,4 v24 q0,4 -4,4
            h-24 q-4,0 -4,-4 v-24 q0,-4 4,-4
            M2,20 h20 v-10 z"/>

在刚才的基础上加上M2,20 h20 v-10 z ，这是一个逆时针方向的三角形，会变成下面这样

![](http://upload-images.jianshu.io/upload_images/1175492-5002566d79577e1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也就是说，我们只需要画一个逆时针方向的心型，就会出现上面的那个效果。先看看代码

    <path
        android:fillColor="#f24e4e"
        android:pathData="M8,4
            h24 q4,0 4,4 v24 q0,4 -4,4
            h-24 q-4,0 -4,-4 v-24 q0,-4 4,-4
            M20,15
            a5,6 -15 0 0 -9,2
            c0,5 4,6 9,12
            c5,-6 9,-7 9,-12
            a5,6 15 0 0 -9 -2"/>

这个是在正方形的基础上加上

        M20,15
        a5,6 -15 0 0 -9,2
        c0,5 4,6 9,12
        c5,-6 9,-7 9,-12
        a5,6 15 0 0 -9 -2

我们看看如果pathData 单独设为上述路径会是什么样的

![](http://upload-images.jianshu.io/upload_images/1175492-c8286ca7cfdc543c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

果然很漂亮。首先它从白色的点 (20,15)开始，向逆时针方向，也就是右边画了一段圆弧到蓝色的点(11,17)，圆弧所在的椭圆逆时针转15°，取短弧，逆时针。然后再画一条三阶贝塞尔曲线，控制点分别是黄点(11,22)，黑点(15,23)，终点是绿色的点(20, 29)。这样子就完成了心型的一半了。另一半跟这边的完全对称，也就不细说了。

完整代码

    <vector xmlns:android="http://schemas.android.com/apk/res/android"
            android:width="24dp"
            android:height="24dp"
            android:viewportWidth="40"
            android:viewportHeight="40">
    
            <path
                android:fillColor="#f24e4e"
                android:pathData="M8,4
                h24 q4,0 4,4 v24 q0,4 -4,4
                h-24 q-4,0 -4,-4 v-24 q0,-4 4,-4
                M20,15
                a5,6 -15 0 0 -9,2
                c0,5 4,6 9,12
                c5,-6 9,-7 9,-12
                a5,6 15 0 0 -9 -2"/>
    </vector>

## 结尾 ##

- Bitmap的绘制效率并不一定会比Vector高，它们有一定的平衡点，当Vector比较简单时，其效率是一定比Bitmap高的，所以，为了保证Vector的高效率，Vector需要更加简单，PathData更加标准、精简，当Vector图像变得非常复杂时，就需要使用Bitmap来代替了
- Vector适用于ICON、Button、ImageView的图标等小的ICON，或者是需要的动画效果，由于Bitmap在GPU中有缓存功能，而Vector并没有，所以Vector图像不能做频繁的重绘
- Vector图像过于复杂时，不仅仅要注意绘制效率，初始化效率也是需要考虑的重要因素





