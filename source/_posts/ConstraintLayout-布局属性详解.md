---
title: ConstraintLayout 布局属性详解
comments: false
date: 2018-05-18 09:31:28
tags: ConstraintLayout
categories:
    - 学习知识
    - ConstraintLayout
    
---

### 简介: ###

ConstraintLayout 翻译为 约束布局，也有人把它称作 增强型的相对布局，由 2016 年 Google I/O 推出。扁平式的布局方式，无任何嵌套，减少布局的层级，优化渲染性能。从支持力度而言，将成为主流布局样式，完全代替其他布局。

<!-- more -->

本文参考了 : [ConstraintLayout 的使用](https://www.jianshu.com/p/6e5fa646ddf5 "ConstraintLayout 的使用")

# 1, 使用 #

ConstraintLayout 是 Android support 中的一个重要控件, 在使用 Android Studio创建项目的时候会自动引用该库 , 在gradle中引用为

    implementation 'com.android.support.constraint:constraint-layout:1.0.2' 

在 [https://developer.android.com/training/constraint-layout/](https://developer.android.com/training/constraint-layout/) 中, 可以看到

![](http://zhaozehui.cn/images/blogiamges/image_2.png)

# 2, 详细属性 #

ConstraintLayout 布局属性可以说是有54种属性, 大致分为8类

### 1, ConstraintLayout 本身使用的属性 (4种) ###

    android_maxHeight
    android_maxWidth
    android_minHeight
    android_minWidth

这四个属性就是 Android 里面常见的控制 View 最大尺寸和最小尺寸的属性，可以设置到 ConstraintLayout 上来控制 ConstraintLayout 的尺寸信息

### 2, 相对定位属性(13种) ###

    layout_constraintBaseline_toBaselineOf		当前控件与目标控件的基线对齐

	layout_constraintTop_toBottomOf				当前上部 与 目标底部 对齐
	layout_constraintTop_toTopOf				上部  	上部
	layout_constraintBottom_toBottomOf			下部	下部
	layout_constraintBottom_toTopOf				下部	上部
	layout_constraintStart_toEndOf
	layout_constraintStart_toStartOf
	layout_constraintEnd_toEndOf
	layout_constraintEnd_toStartOf
	layout_constraintLeft_toLeftOf
	layout_constraintLeft_toRightOf
	layout_constraintRight_toLeftOf
	layout_constraintRight_toRightOf

>这些是用来控制子 View 相对位置的，这些属性和 RelativeLayout 的布局属性非常类似，用来控制子 View 的某一个属性相对于另外一个 View 或者 父容器的位置。 理解为 当前位置的 xxx(constraintXXX) 与 目标的 xxx(toXXOf) 对齐

![](https://upload-images.jianshu.io/upload_images/4179925-929ddcbf1221bc00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

通过 ConstraintLayout 实现的代码如下所示：

    <?xml version="1.0" encoding="utf-8"?>
    <android.support.constraint.ConstraintLayout
        ...
        >
    
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            ...
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"/>
    
        <TextView
            android:id="@+id/tv_a"
            ...
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toLeftOf="@+id/tv_b"
            app:layout_constraintTop_toBottomOf="@+id/toolbar"/>
    
        <TextView
            android:id="@+id/tv_b"
            ...
            app:layout_constraintLeft_toRightOf="@+id/tv_a"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/toolbar"/>
    </android.support.constraint.ConstraintLayout>

### 3, Margin （View 的边距）(12种) ###

    普通的 view控制margin
	– layout_marginStart
	– layout_marginEnd
	– layout_marginLeft
	– layout_marginTop
	– layout_marginRight
	– layout_marginBottom
	margin 值只能为大于等于0的数字，这些都是很常见的属性

	– layout_goneMarginBottom
	– layout_goneMarginEnd
	– layout_goneMarginLeft
	– layout_goneMarginRight
	– layout_goneMarginStart
	– layout_goneMarginTop
>这6个是控制当前 View 所参考的 View 状态为 GONE 的时候的 margin 值.(这个值不常用, 但是万一需要用的时候就很方便了)
>
>比如 ButtonB 左边相对于 ButtonA 右边对齐，ButtonA 左边相对于父容器左边对齐。如果 ButtonA 的状态为 GONE（不可见的），则 ButtonB 就相对于父容器左边对齐了。如果有个需求是，当 ButtonA 不可见的时候， ButtonB 和父容器左边需要一个边距 16dp。 这个时候就需要使用上面的 layout_goneMarginLeft 或者 layout_goneMarginStart 属性了，如果设置了这个属性，当 ButtonB 所参考的 ButtonA 可见的时候，这个边距属性不起作用；当 ButtonA 不可见（GONE）的时候，则这个边距就在 ButtonB 上面起作用了。
>
>另外还有一个用途就是方便做 View 动画，可以先设置 ButtonA 为 GONE，同时可以保持 ButtonB 的布局位置不变。

### 4, 居中和偏移(bias)(2种) ###

**这两个属性 默认 是 0.5 的, 所以一般的当左边和右边拉力一致的时候是不会改变的**

- layout_constraintHorizontal_bias		横向的权重 , 从左到右 为 0.0 - 1.0
- layout_constraintVertical_bias			纵向的权重 , 从上到下 为 0.0 - 1.0

这个偏移是指左边或者上边的占比,类似于Linearlayout的wedgit属性,区别是wedgit属性必须指定左右两边的占比,但是使用 bias 只需要指定一边即可, 因为权重是从0.0 - 1.0 变化的, 指定一边权重后另一边的权重即为 1-bias.

### 5, 子 View 的尺寸控制(14种) ###

- android:layout_width		子view的宽 , 三种取值 1,确定的尺寸，比如 48dp; 2,WRAP_CONTENT ，和其他地方的 WRAP_CONTENT 一样 3, 0dp，这个选项等于 “MATCH_CONSTRAINT”
>注意： MATCH_PARENT 属性无法在 ConstraintLayout 里面的 子 View 上使用。

- 控制子 View 的宽高比 layout_constraintDimensionRatio

        layout_constraintWidth_default
    	layout_constraintHeight_default		取值为 spread 或者 wrap，默认值为 spread ，占用所有的符合约束的空间；如果取值为 Wrap ，并且view 的尺寸设置为 wrap_content 且受所设置的约束限制其尺寸，则 这个 view 最终尺寸不会超出约束的范围。
    	layout_constraintHeight_max 		取值为具体的尺寸
    	layout_constraintHeight_min 		取值为具体的尺寸
    	layout_constraintWidth_max 			取值为具体的尺寸
    	layout_constraintWidth_min 			取值为具体的尺寸

其中: layout_constraintDimensionRatio 非常实用

        <android.support.constraint.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <Button
                android:id="@+id/btn_1"
                android:layout_width="0dp"
                android:layout_height="0dp"
                android:text="constraint"
                toolbar:layout_constraintBottom_toBottomOf="parent"
                toolbar:layout_constraintDimensionRatio="h,2:1"
                toolbar:layout_constraintLeft_toLeftOf="parent"
                toolbar:layout_constraintRight_toRightOf="parent"
                toolbar:layout_constraintTop_toTopOf="parent" />
        </android.support.constraint.ConstraintLayout>

上面的view中共有四个约束, 即button的宽高都是0dp, 配合四个 parent 的约束使得其宽高全部撑满的父布局, 使用了  layout_constraintDimensionRatio="h,2:1" 后形成了 如下效果:
![](http://zhaozehui.cn/images/blogiamges/image_3.png)
宽撑满, 竖直方向居中 


如果使用 layout_constraintDimensionRatio="w,1:2" , 则会形成另一种效果:
![](http://zhaozehui.cn/images/blogiamges/image_4.png)
高撑满, 水平方向居中

适合在那种需求中实现呢:  在一个布局中, 如果一个控件宽或者高撑满父布局, 宽高需要成比例,但是整个布局需要在父布局居中就需要这样了.
关于 layout_constraintDimensionRatio 后的 w,h的解释为:

- 默认为 宽比高  即w:h
- w, 在前或者 h, 在前的区别, 首先明白 constraint的布局为约束, 所以 前面是w  即对w 的约束. 上面说 默认的时候 为 w:h 是对 h 的约束.所以 上图使用 h,2:1 ,宽撑满,高度为被约束后的 1/2 宽的值.   而 w,1:2 则是对宽的约束, 即高撑满, 宽为 1/2 高的值

### 6, 链条布局（Chains） ###
Chains 为同一个方向（水平或者垂直）上的多个子 View 提供一个类似群组的概念。其他的方向则可以单独控制。

Chain 链是一种特殊的约束让多个 chain 链连接的 Views 能够平分剩余空间位置。在 Android 传统布局特性里面最相似的应该是 LinearLayout 中的权重比 weight ，但 Chains 链能做到的远远不止权重比 weight 的功能。

- layout_constraintHorizontal_chainStyle
- layout_constraintHorizontal_weight
- layout_constraintVertical_chainStyle
- layout_constraintVertical_weight

- spread 模式（Chain 链的默认模式）——它将平分间隙让多个 Views 布局到剩余空间
- Spread Inside Chain 链模式——它将会把两边最边缘的两个 View 到外向父组件边缘的距离去除，然后让剩余的 Views 在剩余的空间内平分间隙布局
- Spread 系的权重, spread 和 spread inside Chain 链可以设置每个组件的 weight 权重，这跟 LinearLayout 的 weight 权重设置很像。
- Packed Chain 链模式——它将所有 Views 打包到一起不分配多余的间隙（当然不包括通过 margin 设置多个 Views 之间的间隙），然后将整个组件组在可用的剩余位置居中

![](https://upload-images.jianshu.io/upload_images/2647955-2c8dd3fef1318925.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

其中Weighted Chain链的默认是在可用空间中平均分配元素。如果一个或多个元素使用MATCH_CONSTRAINT，它们将使用可用的空白空间（在它们之间平分）。属性layout_constraintHorizo​​ntal_weight和layout_constraintVertical_weight将控制如何使用MATCH_CONSTRAINT在元素之间分配空间。例如，在使用MATCH_CONSTRAINT的包含两个元素的链上，第一个元素使用权重为2，第二个元素的权重为1，第一个元素占用的空间将是第二个元素的两倍。例如要把A、B、C按钮水平排成一行，可以用链式约束

### 7,UI 编辑器所使用的属性 ###

下面几个属性是 UI 编辑器所使用的，用了辅助拖拽布局的，在实际使用过程中，可以不用关心这些属性

    layout_optimizationLevel
	layout_editor_absoluteX
	layout_editor_absoluteY
	layout_constraintBaseline_creator
	layout_constraintTop_creator
	layout_constraintRight_creator
	layout_constraintLeft_creator
	layout_constraintBottom_creator

# 2, Guideline #
在ConstraintLayout中有一类对象，在运行的时候不显示任何UI效果，只是作为参照物辅助布局，GuideLine就是其中之一。GuideLine分为水平引导线和垂直引导线。

它的属性有四种: 

    android_orientation 				控制 Guideline 是横向的还是纵向的
	layout_constraintGuide_begin 		控制 Guideline 距离父容器开始的距离	,使用dp表示,是一个长度
	layout_constraintGuide_end 			控制 Guideline 距离父容器末尾的距离	
	layout_constraintGuide_percent 		控制 Guideline 在父容器中的位置为百分比	,使用 0.0-1.0的float表示, 超过1.0按照1.0算

例如: 

     android:orientation="horzontal"
     app:layout_constraintGuide_begin="50dp"
     app:layout_constraintGuide_end="50dp"
     app:layout_constraintGuide_percent="0.5"

>可以使用layout_constraintGuide_begin和layout_constraintGuide_end设置具体dp值，也可以使用layout_constraintGuide_percent来设置比例。实际上Guideline 类其实就是一个 View，而且它不会渲染任何东西因为它实现了一个 final 的 onDraw() 而且固定了它的可见性为 View.GONE ，这就决定了运行时不会显示任何东西，而在 View 的 layout 布局过程中它会占据一个位置，而其他组件可以通过它来布局对齐。所以实际上的 Guideline 只是一个极其轻量级没有任何显示但是可以用于约束布局对齐的 View 组件，当我们以创建从 view 的一个锚点到参照线的约束 constraint 对象来根据参照线来对齐这个 view时，参照线移动时，受约束的 view 也会跟着参照线一起移动，最后，参照线 Guideline 拥有了一个属性 app:orientation="vertical" 来描述它是一个垂直的参照线（此处也可以设置为 horizontal）。它还有属性app:layout_constraintGuide_begin="16dp" 来描述它是一个对齐父组件的 start 边缘的 16dp 偏移量处。再次提醒的是，应该用 start 边缘而不是 left 边缘。当然切换向 end 类型的话，可以使用另一个属性 app:layout_constraintGuide_end="..." ，切换为百分比类型的参照线则是设置属性 app:layout_constraintGuide_percent="0.5" 值得取值范围为 0.0 到 1.0 ，描述的是百分比偏移量。

# 3,通过代码来设置 ConstraintLayout 属性(ConstraintSet 支持所有属性设置) #

XML 布局文件中使用的属性，只能在 XML 布局文件中使用，但是现在针对 UI 做动画的时候，需要通过代码来动态设置 View 的布局属性。

ConstraintSet 是用来通过代码管理布局属性的集合对象，可以通过这个类来创建各种布局约束，然后把创建好的布局约束应用到一个 ConstraintLayout 上，可以通过如下几种方式来创建 ConstraintSet：

手工创建：

	c = new ConstraintSet(); c.connect(….);
	//从 R.layout.* 对象获取
	c.clone(context, R.layout.layout1);
	//从 ConstraintLayout 中获取
	c.clone(clayout);


然后通过 applyTo 函数来应用到ConstraintLayout 上

	mConstraintSet.applyTo(mConstraintLayout); // set new constraints











