---
title: Android动画详细解读
comments: false
date: 2018-05-23 14:37:01
tags: Android动画
categories:
    - Android
    - 知识总结

---

出处: [https://blog.csdn.net/m0_37700275/article/details/78690243](https://blog.csdn.net/m0_37700275/article/details/78690243)

# Android动画类型分类 #

1. 逐帧动画【Frame Animation】，即顺序播放事先做好的图像，跟电影类似 
2. 补间动画【Tween Animation】，即通过对场景里的对象不断做图像变换 ( 平移、缩放、旋转 ) 产生动画效果 
3. 属性动画【Property Animation】，补间动画增强版，支持对对象执行动画 
4. 过渡动画【Transition Animation】,实现Activity或View过渡动画效果

<!-- more -->

# Android动画实现方式分类 #

- XML资源文件 
- 代码方式

# Android动画发展史 #

- Android 3.0之前版本，逐帧动画，补间动画 
- Android 3.0之后版本，属性动画 
- Android 4.4中，过渡动画

# Android动画详解 #

## Android逐帧动画 ##

### 逐帧动画简单介绍 ###

也叫Drawable Animation动画，是最简单最直观动画类型

### 逐帧动画XML资源文件方式 ###

这个是最常用的方式，在res/drawable目录下新建动画XML文件，如下所示 

- android:oneshot用来控制动画是否循环播放，true表示不会循环播放，false表示会循环播放 
- android:duration=”200”表示每一帧持续播放的时间


        <?xml version="1.0" encoding="utf-8"?>
        <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
            android:oneshot="false">
    
            <item android:drawable="@drawable/refresh_07" android:duration="20" />
            <item android:drawable="@drawable/refresh_08" android:duration="20" />
            <item android:drawable="@drawable/refresh_09" android:duration="20" />
            <item android:drawable="@drawable/refresh_10" android:duration="20" />
        
        </animation-list>


### 逐帧动画代码方式 ###

代码方式一般不使用.

    AnimationDrawable drawable = new AnimationDrawable();
    for(int a=0 ; a<9 ; a++){
        int id = getResources().getIdentifier("audio_anim_0" + a, "mipmap", getPackageName());
        Drawable da = getResources().getDrawable(id);
        drawable.addFrame(da,200);
    }
    ivVisualEffect.setBackground(drawable);
    drawable.setOneShot(false);
    //获取对象实例，用来控制播放与停止
    AnimationDrawable rocketAnimation = (AnimationDrawable) ivVisualEffect.getBackground();
    rocketAnimation.start();    // 开启帧动画
    rocketAnimation.stop();     // 停止动画

使用方式:

    <ImageView
        android:id="@+id/iv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/loading"
        android:layout_centerInParent="true" />
    或者: <!-- android:background="@drawable/loading" -->

最后

    Drawable drawable = iv.getDrawable();
    //如果是用的是 background 的方式: Drawable background = iv.getBackground();
    if (background instanceof AnimationDrawable) {
        AnimationDrawable rocketAnimation = (AnimationDrawable) background;
        rocketAnimation.start();
    }

## Android补间动画 ##

### 补间动画简单介绍 ###

无需关注每一帧，只需要定义动画开始与结束两个关键帧，并指定动画变化的时间与方式等 。主要有四种基本的效果 

- 透明度变化 
- 大小缩放变化 
- 位移变化 
- 旋转变化 

**表现形式**

#### **XML中** ####

- alph                   渐变透明度动画效果
- scale                  渐变尺寸伸缩动画效果
- translate              画面转换位置移动动画效果
- rotate                 画面转移旋转动画效果

#### **JavaCode中** ####
- AlphaAnimation         渐变透明度动画效果
- ScaleAnimation         渐变尺寸伸缩动画效果
- TranslateAnimation     画面转换位置移动动画效果
- RotateAnimation        画面转移旋转动画效果

#### 插值器 ####

Android系统会在补间动画开始和结束关键帧之间插入渐变值，它依据插值器。 

Interpolator 时间插值类，定义动画变换的速度。能够实现alpha/scale/translate/rotate动画的加速、减速和重复等。Interpolator类其实是一个空接口，继承自TimeInterpolator，TimeInterpolator时间插值器允许动画进行非线性运动变换，如加速和限速等，该接口中只有接口中有一个方法 float getInterpolation(float input)这个方法。传入的值是一个0.0~1.0的值，返回值可以小于0.0也可以大于1.0。 

#### Android内置的插值器(10个) ####

- LinearInterpolator 线性，线性均匀改变
- AccelerateInterpolator 加速，开始时慢中间加速
- DecelerateInterpolator 减速，开始时快然后减速
- AccelerateDecelerateInterolator 先加速后减速，开始结束时慢，中间加速
- AnticipateInterpolator 反向，先向相反方向改变一段再加速播放
- OvershootInterpolator超越，最后超出目的值然后缓慢改变到目的值
- AnticipateOvershootInterpolator 反向加超越，先向相反方向改变，再加速播放，会超出目的值然后缓慢移动至目的值
- BounceInterpolator 跳跃，快到目的值时值会跳跃，如目的值100，后面的值可能依次为85，77，70，80，90，100
- CycleIinterpolator 循环，动画循环一定次数，值的改变为一正弦函数：Math.sin(2* mCycles* Math.PI* input)
- PathInterpolator新增的，就是可以定义路径坐标，然后可以按照路径坐标来跑动；注意其坐标并不是 XY，而是单方向，也就是我可以从0~1，然后弹回0.5 然后又弹到0.7 有到0.3，直到最后时间结束。

#### PS: ####

 Android 3.0之前版本的补间动画位于 sdk中 android.jar 包下的 android/view/animation 中, 这个包中包含了所有关于补间动画的类和工具 其中 ;

- AnimationUtils 用于读取xml中设置的 anim.
- Interpolator 用于补间动画中的插值器
- 各种实现 Interpolator 的具体效果插值器
- 以及四大 补间动画 

Android 3.0之后的属性动画 位于 sdk中 android.jar 包新增加的 animation包下(这个包是3.0之前没有的), 这个包中主要包含了属性动画的类

- 以Animator为首
- TimeInterpolator ,属性动画以及补间动画插值器的最终接口 ,插值器
- TypeEvaluator , 估值器 , 使用TypeEvaluator根据插值因子计算属性值，Android系统可识别的类型包括int、float和颜色，分别由 IntEvaluator、 FloatEvaluator、 ArgbEvaluator 提供支持 此外新版本中还添加了 intarray 和float array等
- ValueAnimator ,继承自 Animator ,主要通过回调来主动进行动画
- TimeAnimator / ObjectAnimator ,继承自 ValueAnimator ,被动进行动画, 即自动进行动画

关于 插值器和估值器可以查看 [模拟自然动画的精髓——TimeInterpolator与TypeEvaluator](https://www.jianshu.com/p/b239d14060a8)

### AplhaAnimation ###

第一种方式：XML方式

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android" > 

        <alpha 
            android:interpolator="@android:anim/accelerate_decelerate_interpolator" 
            android:duration="1000" 
            android:fromAlpha="0.0" 
            android:toAlpha="1.0" /> 
        <!-- 
        透明度控制动画效果 alpha 
        浮点型值： 
        fromAlpha 属性为动画起始时透明度 
        toAlpha   属性为动画结束时透明度 
        说明: 
        0.0表示完全透明 
        1.0表示完全不透明 
        以上值取0.0-1.0之间的float数据类型的数字 
        长整型值： 
        duration  属性为动画持续时间 
        说明:时间以毫秒为单位 
        --> 
    </set>

第二种方式：代码方式

    AlphaAnimation alpha = new AlphaAnimation(0, 1); 
    alpha.setDuration(500);          //设置持续时间 
    alpha.setFillAfter(true);                   //动画结束后保留结束状态 
    alpha.setInterpolator(new AccelerateInterpolator());        //添加插值器 
    ivImage.setAnimation(alpha);

### ScaleAnimation ###

第一种方式：XML方式

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android" > 
        <scale 
            android:duration="1000" 
            android:fillAfter="false" 
            android:fromXScale="0.0" 
            android:fromYScale="0.0" 
            android:interpolator="@android:anim/accelerate_decelerate_interpolator" 
            android:pivotX="50%" 
            android:pivotY="50%" 
            android:toXScale="1.4" 
            android:toYScale="1.4" /> 
        <!-- 
        尺寸伸缩动画效果 scale 
        属性：interpolator 指定一个动画的插入器 
        在我试验过程中，使用android.res.anim中的资源时候发现 
        有三种动画插入器: 
        accelerate_decelerate_interpolator  加速-减速 动画插入器 
        accelerate_interpolator             加速-动画插入器 
        decelerate_interpolator             减速- 动画插入器 
        其他的属于特定的动画效果 
        浮点型值： 
        fromXScale 属性为动画起始时 X坐标上的伸缩尺寸 
        toXScale   属性为动画结束时 X坐标上的伸缩尺寸 
        fromYScale 属性为动画起始时Y坐标上的伸缩尺寸 
        toYScale   属性为动画结束时Y坐标上的伸缩尺寸 
        说明: 
        以上四种属性值 
        0.0表示收缩到没有 
        1.0表示正常无伸缩 
        值小于1.0表示收缩 
        值大于1.0表示放大 
        pivotX     属性为动画相对于物件的X坐标的开始位置 
        pivotY     属性为动画相对于物件的Y坐标的开始位置 
        说明: 
        以上两个属性值 从0%-100%中取值 
        50%为物件的X或Y方向坐标上的中点位置 
        长整型值： 
        duration  属性为动画持续时间 
        说明:   时间以毫秒为单位 
        布尔型值: 
        fillAfter 属性 当设置为true ，该动画转化在动画结束后被应用 
        --> 
    </set>

第二种方式：代码方式

    ScaleAnimation scale = new ScaleAnimation(1.0f, scaleXY, 1.0f, scaleXY, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f); 
    scale.setDuration(durationMillis); 
    scale.setFillAfter(true); 
    ivImage.setAnimation(scale);

### TranslateAnimation ###

第一种方式：XML资源文件方式

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android" > 
        <translate 
            android:duration="2000" 
            android:fromXDelta="30" 
            android:fromYDelta="30" 
            android:toXDelta="-80" 
            android:toYDelta="300" /> 
        <!-- 
        translate 位置转移动画效果 
        整型值: 
        fromXDelta 属性为动画起始时 X坐标上的位置 
        toXDelta   属性为动画结束时 X坐标上的位置 
        fromYDelta 属性为动画起始时 Y坐标上的位置 
        toYDelta   属性为动画结束时 Y坐标上的位置 
        注意: 
         没有指定fromXType toXType fromYType toYType 时候， 
         默认是以自己为相对参照物 
        长整型值： 
        duration  属性为动画持续时间 
        说明:   时间以毫秒为单位 
        --> 
    </set>

第二种方式：代码方式

    TranslateAnimation translate = new TranslateAnimation(fromXDelta, toXDelta, fromYDelta, toYDelta); 
    translate.setDuration(durationMillis); 
    translate.setFillAfter(true); 
    ivImage.setAnimation(translate);

### RotateAnimation ###

第一种方式：XML资源文件方式

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android" > 
        <rotate 
            android:duration="3000" 
            android:fromDegrees="0" 
            android:interpolator="@android:anim/accelerate_decelerate_interpolator" 
            android:pivotX="50%" 
            android:pivotY="50%" 
            android:toDegrees="+350" /> 
        <!-- 
        rotate 旋转动画效果 
        属性：interpolator 指定一个动画的插入器 
        浮点数型值: 
        fromDegrees 属性为动画起始时物件的角度 
        toDegrees  属性为动画结束时物件旋转的角度 可以大于360度 
        说明: 
        当角度为负数-表示逆时针旋转 
        当角度为正数-表示顺时针旋转 
        (负数from-to正数:顺时针旋转) 
        (负数from-to负数:逆时针旋转) 
        (正数from-to正数:顺时针旋转) 
        (正数from-to负数:逆时针旋转) 
    
        pivotX    属性为动画相对于物件的X坐标的开始位置 
        pivotY    属性为动画相对于物件的Y坐标的开始位置 
    
        说明:        以上两个属性值 从0%-100%中取值    50%为物件的X或Y方向坐标上的中点位置 
        长整型值： 
        duration  属性为动画持续时间 
        说明:      时间以毫秒为单位 
        --> 
    </set>

第二种方式：代码方式

    RotateAnimation rotate = new RotateAnimation(fromDegrees, toDegrees, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f); 
    rotate.setDuration(durationMillis); 
    rotate.setFillAfter(true); 
    ivImage.setAnimation(rotate);

### PS2 ###

一般的,创建xml文件形式的方式更将简单方便. 但是创建xml的时候的标签对动画效果是非常有影响的,其中 translate/alpha/rotate/scale是动画本身,可以方便的设置 repeatCount 和 repeatMode 信息, 但是当标签是 set 的时候,则load以后的 Animation 本质上是一个 AnimationSet ,这个 AnimationSet 中,如果想要让其中的元素无限循环显示(即 设置repeatCount为 -1)那么是没有效果的, 动画只会执行一遍,所以在使用set 的时候如果需要无限播放动画 需要在 set中的子标签(具体的四种动画标签, 不算set)需要自行设置 
           
    android:repeatCount="-1"
    android:repeatMode="reverse"

## Android属性动画 ##

> 注意，以XML方式，res的文件夹名称必须是animator，否则无法引用

### 属性动画基本介绍 ###

补间动画增强版本，补间动画存在一些缺点 

- 作用对象局限：View 。即补间动画 只能够作用在视图View上，即只可以对一个Button、TextView、甚至是LinearLayout、或者其它继承自View的组件进行动画操作，但无法对非View的对象进行动画操作 
- 没有改变View的属性，只是改变视觉效果 
- 动画效果单一

#### 属性动画特点  ####

作用对象：任意 Java 对象，不再局限于 视图View对象 

实现的动画效果：可自定义各种动画效果，不再局限于4种基本变换：平移、旋转、缩放 & 透明度 

#### 基本工作原理  ####

在一定时间间隔内，通过不断对值进行改变，并不断将该值赋给对象的属性，从而实现该对象在该属性上的动画效果 

属性动画基类：Animator，抽象类。子类有两个重要的类：ValueAnimator 类 & ObjectAnimator 类，其他类：Evaluator类，AnimatorSet类 

关于常用属性动画类总结如下图所示 :

![](http://upload-images.jianshu.io/upload_images/4432347-b5cf6a8b0e450a01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Evaluator ####

**估值器（TypeEvaluator）作用**

- 设置动画 如何从初始值过渡到结束值的逻辑 
- 插值器（Interpolator）决定值的变化模式（匀速、加速blabla） 
- 估值器（TypeEvaluator）决定值的具体变化数值 

##### 看看接口TypeEvaluator 源代码 #####

    public interface TypeEvaluator<T> { 
        public T evaluate(float fraction, T startValue, T endValue); 
    }

##### 看看如何实现估值器 #####

    public class ArgbEvaluator implements TypeEvaluator { 
        private static final ArgbEvaluator sInstance = new ArgbEvaluator(); 
        public static ArgbEvaluator getInstance() { 
            return sInstance; 
        } 
        // FloatEvaluator实现了TypeEvaluator接口 
        public Object evaluate(float fraction, Object startValue, Object endValue) { 
            // 参数说明 
            // fraction：表示动画完成度（根据它来计算当前动画的值） 
            // startValue、endValue：动画的初始值和结束值 
            int startInt = (Integer) startValue; 
            int startA = (startInt >> 24) & 0xff; 
            int startR = (startInt >> 16) & 0xff; 
            int startG = (startInt >> 8) & 0xff; 
            int startB = startInt & 0xff; 
            int endInt = (Integer) endValue; 
            int endA = (endInt >> 24) & 0xff; 
            int endR = (endInt >> 16) & 0xff; 
            int endG = (endInt >> 8) & 0xff; 
            int endB = endInt & 0xff; 
            // 初始值 过渡 到结束值 的算法是： 
            // 1. 用结束值减去初始值，算出它们之间的差值 
            // 2. 用上述差值乘以fraction系数 
            // 3. 再加上初始值，就得到当前动画的值 
            return (int)((startA + (int)(fraction * (endA - startA))) << 24) | 
                    (int)((startR + (int)(fraction * (endR - startR))) << 16) | 
                    (int)((startG + (int)(fraction * (endG - startG))) << 8) | 
                    (int)((startB + (int)(fraction * (endB - startB)))); 
        } 
    }

代码中:

    public static ValueAnimator ofArgb(int... values) { 
        ValueAnimator anim = new ValueAnimator(); 
        anim.setIntValues(values); 
        anim.setEvaluator(ArgbEvaluator.getInstance()); 
        return anim; 
    } 


#### AnimatorSet ####

##### 特点  #####

单一动画实现的效果相当有限，更多的使用场景是同时使用多种动画效果，即组合动画 

PS:Animator的xml读取方式和 animation不同, animation读取方式是通过 AnimationUtils.load()方式获取到 Animation ,而Animator读取xml是通过 AnimatorInflater.loadAnimator的方式获取 Animator.

第一种方式：xml方式

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android" 
        android:ordering="sequentially" > 
        <!--表示Set集合内的动画按顺序进行--> 
        <!--ordering的属性值:sequentially & together--> 
        <!--sequentially:表示set中的动画，按照先后顺序逐步进行（a 完成之后进行 b ）--> 
        <!--together:表示set中的动画，在同一时间同时进行,为默认值--> 
    
        <set android:ordering="together" > 
            <!--下面的动画同时进行--> 
            <objectAnimator 
                android:duration="2000" 
                android:propertyName="translationX" 
                android:valueFrom="0" 
                android:valueTo="300" 
                android:valueType="floatType" > 
            </objectAnimator> 
    
            <objectAnimator 
                android:duration="3000" 
                android:propertyName="rotation" 
                android:valueFrom="0" 
                android:valueTo="360" 
                android:valueType="floatType" > 
            </objectAnimator> 
        </set> 
    
        <set android:ordering="sequentially" > 
            <!--下面的动画按序进行--> 
            <objectAnimator 
                android:duration="1500" 
                android:propertyName="alpha" 
                android:valueFrom="1" 
                android:valueTo="0" 
                android:valueType="floatType" > 
            </objectAnimator> 
            <objectAnimator 
                android:duration="1500" 
                android:propertyName="alpha" 
                android:valueFrom="0" 
                android:valueTo="1" 
                android:valueType="floatType" > 
            </objectAnimator> 
        </set>
     </set> 

第二种方式：Java方式

    // 步骤1：设置需要组合的动画效果 
    ObjectAnimator translation = ObjectAnimator.ofFloat(mButton, "translationX", curTranslationX, 300,curTranslationX);  // 平移动画 
    ObjectAnimator rotate = ObjectAnimator.ofFloat(mButton, "rotation", 0f, 360f);  // 旋转动画 
    ObjectAnimator alpha = ObjectAnimator.ofFloat(mButton, "alpha", 1f, 0f, 1f);  // 透明度动画 // 步骤2：创建组合动画的对象 
    AnimatorSet animSet = new AnimatorSet();  // 步骤3：根据需求组合动画 
    animSet.play(translation).with(rotate).before(alpha);  
    animSet.setDuration(5000);  // 步骤4：启动动画 
    animSet.start();

常用方法:

- AnimatorSet.play(Animator anim)   ：播放当前动画
- AnimatorSet.after(long delay)   ：将现有动画延迟x毫秒后执行
- AnimatorSet.with(Animator anim)   ：将现有动画和传入的动画同时执行
- AnimatorSet.after(Animator anim)   ：将现有动画插入到传入的动画之后执行
- AnimatorSet.before(Animator anim) ：  将现有动画插入到传入的动画之前执行

#### ValueAnimator ####

**基本作用**

将初始值 以整型数值的形式 过渡到结束值 。即估值器是整型估值器 - IntEvaluator 

- ValueAnimator.oFloat（）采用默认的浮点型估值器 (FloatEvaluator) 
- ValueAnimator.ofInt（）采用默认的整型估值器（IntEvaluator） 

第一种实现方式：Java设置

    public static ValueAnimator setValueAnimator(View view , int start , int end , int time , int delay , int count){ 
        // 步骤1：设置动画属性的初始值 & 结束值 
        ValueAnimator mAnimator = ValueAnimator.ofInt(start, end); 
        // ofInt（）作用有两个 
        // 1. 创建动画实例 
        // 2. 将传入的多个Int参数进行平滑过渡:此处传入0和1,表示将值从0平滑过渡到1 
        // 如果传入了3个Int参数 a,b,c ,则是先从a平滑过渡到b,再从b平滑过渡到C，以此类推 
        // ValueAnimator.ofInt()内置了整型估值器,直接采用默认的.不需要设置，即默认设置了如何从初始值 过渡到 结束值 
        // 下面看看ofInt()的源码分析 ->>关注1 
        mAnimator.setTarget(view); 
    
        // 步骤2：设置动画的播放各种属性 
        mAnimator.setDuration(time); 
        // 设置动画运行的时长 
    
        mAnimator.setStartDelay(delay); 
        // 设置动画延迟播放时间 
    
        mAnimator.setRepeatCount(count); 
        // 设置动画重复播放次数 = 重放次数+1 
        // 动画播放次数 = infinite时,动画无限重复 
    
        mAnimator.setRepeatMode(ValueAnimator.RESTART); 
        // 设置重复播放动画模式 
        // ValueAnimator.RESTART(默认):正序重放 
        // ValueAnimator.REVERSE:倒序回放 
    
        // 步骤3：将改变的值手动赋值给对象的属性值：通过动画的更新监听器 
        // 设置 值的更新监听器 
        // 即：值每次改变、变化一次,该方法就会被调用一次 
        return mAnimator; 
    } 
    //------- 
    Button b1 = (Button) findViewById(R.id.b1); 
    ValueAnimator valueAnimator = AnimatorUtils.setValueAnimator(b1,0, 2, 2000, 500, 2); 
    valueAnimator.start();

第二种实现方式：XML设置

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android"> 
        <animator 
            android:valueFrom="0" 
            android:valueTo="100" 
            android:valueType="intType" 
            android:duration="3000" 
            android:startOffset ="1000" 
            android:fillBefore = "true" 
            android:fillAfter = "false" 
            android:fillEnabled= "true" 
            android:repeatMode= "restart" 
            android:repeatCount = "0" 
            android:interpolator="@android:anim/accelerate_interpolator"/> 
            <!--初始值--> 
            <!--结束值--> 
            <!--变化值类型 ：floatType & intType--> 
            <!--动画持续时间（ms），必须设置，动画才有效果--> 
            <!--动画延迟开始时间（ms）--> 
            <!--动画播放完后，视图是否会停留在动画开始的状态，默认为true--> 
            <!--动画播放完后，视图是否会停留在动画结束的状态，优先于fillBefore值，默认为false--> 
            <!--是否应用fillBefore值，对fillAfter值无影响，默认为true--> 
            <!--选择重复播放动画模式，restart代表正序重放，reverse代表倒序回放，默认为restart|--> 
            <!--重放次数（所以动画的播放次数=重放次数+1），为infinite时无限重复--> 
            <!--插值器，即影响动画的播放速度,下面会详细讲--> 
    </set> 
    //代码引用 
    Button b3 = (Button) findViewById(R.id.b3); 
    Animator mAnim = AnimatorInflater.loadAnimator(this, R.animator.animator_1_0); 
    mAnim.setTarget(b3); 
    mAnim.start();

#### ObjectAnimator ####

直接对对象的属性值进行改变操作，从而实现动画效果 ,继承自ValueAnimator类，即底层的动画实现机制是基于ValueAnimator类 

第一种实现方式：Java设置

    public static ObjectAnimator setObjectAnimator(View view , String type , int start , int end , long time){ 
        ObjectAnimator mAnimator = ObjectAnimator.ofFloat(view, type, start, end); 
        // ofFloat()作用有两个 
        // 1. 创建动画实例 
        // 2. 参数设置：参数说明如下 
        // Object object：需要操作的对象 
        // String property：需要操作的对象的属性 
        // float ....values：动画初始值 & 结束值（不固定长度） 
        // 若是两个参数a,b，则动画效果则是从属性的a值到b值 
        // 若是三个参数a,b,c，则则动画效果则是从属性的a值到b值再到c值 
        // 以此类推 
        // 至于如何从初始值 过渡到 结束值，同样是由估值器决定，此处ObjectAnimator.ofFloat（）是有系统内置的浮点型估值器FloatEvaluator，同ValueAnimator讲解 
    
        // 设置动画重复播放次数 = 重放次数+1 
        // 动画播放次数 = infinite时,动画无限重复 
        mAnimator.setRepeatCount(ValueAnimator.INFINITE); 
        // 设置动画运行的时长 
        mAnimator.setDuration(time); 
        // 设置动画延迟播放时间 
        mAnimator.setStartDelay(0); 
        // 设置重复播放动画模式 
        mAnimator.setRepeatMode(ValueAnimator.RESTART); 
        // ValueAnimator.RESTART(默认):正序重放 
        // ValueAnimator.REVERSE:倒序回放 
        //设置插值器 
        mAnimator.setInterpolator(new LinearInterpolator()); 
        return mAnimator; 
    }

关于preperty的属性值有，如下表所示

![](http://upload-images.jianshu.io/upload_images/4432347-778bf977092c37a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二种方式：XML方式

    <?xml version="1.0" encoding="utf-8"?> 
    <set xmlns:android="http://schemas.android.com/apk/res/android"> 
        <ObjectAnimator 
            android:valueFrom="1" 
            android:valueTo="0" 
            android:valueType="floatType" 
            android:duration = "800" 
            android:propertyName="alpha"/> 
    </set> 
    Animator mAnim = AnimatorInflater.loadAnimator(this, R.animator.animator_1_0); 
    mAnim.setTarget(fabHomeRandom); 
    mAnim.start();

#### ValueAnimator与ObjectAnimator区别 ####

- ValueAnimator 类是先改变值，然后手动赋值 给对象的属性从而实现动画；是间接对对象属性进行操作； 
- ObjectAnimator 类是先改变值，然后自动赋值 给对象的属性从而实现动画；是直接对对象属性进行操作；
- 上述两点实际上是因为: ObjectAnimator 继承自 ValueAnimator ,所以使用 ObjectAnimator 的时候,会通过父类 ValueAnimator继承的方法获得改变的值, 然后自动赋值. ObjectAnimator是 ValueAnimator某一方面的具象化,在部分情况下方便使用.

#### 监听动画器 ####

Animation类通过监听动画开始 / 结束 / 重复 / 取消时刻来进行一系列操作，如跳转页面等等。通过在Java代码里addListener（）设置，因Animator类、AnimatorSet类、ValueAnimator、ObjectAnimator类存在继承关系，所以AnimatorSet类、ValueAnimator、ObjectAnimator都可以使用addListener()监听器进行动画监听。

    mAnim.addListener(new Animator.AnimatorListener() { 
        @Override 
        public void onAnimationStart(Animator animation) { 
            //动画开始时执行 
        } 
    
        @Override 
        public void onAnimationEnd(Animator animation) { 
            //动画结束时执行 
        } 
    
        @Override 
        public void onAnimationCancel(Animator animation) { 
            //动画取消时执行 
        } 
    
        @Override 
        public void onAnimationRepeat(Animator animation) { 
            //动画重复时执行 
        } 
    });

一般的可以使用 Animator.AnimatorListener 的适配器监听

    mAnim2.addListener(new AnimatorListenerAdapter() { 
        // 向addListener()方法中传入适配器对象AnimatorListenerAdapter() 
        // 由于AnimatorListenerAdapter中已经实现好每个接口 
        // 所以这里不实现全部方法也不会报错 
        @Override 
        public void onAnimationCancel(Animator animation) { 
            super.onAnimationCancel(animation); 
            ToastUtils.showShort("动画结束了"); 
        } 
    });    

### Android动画框架原理解析 ###

要了解Android动画是如何加载出来的,我们首先要了解Android View 是如何组织在一起的.每个窗口是一颗View树. RootView是DecorView,在布局文件中声明的布局都是DecorView的子View.是通过setContentView来设置进入窗口内容的. 因为View的布局就是一棵树.所以绘制的时候也是按照树形结构来遍历每个View进行绘制.ViewRoot.java中 draw函数准备好Canvas后 调用 mView.draw(canvas),这里的mView是DecorView. 

下面看一下递归绘制的几个步骤: 

1. 绘制背景 
2. 如果需要,保存画布(canvas),为淡入淡出做准备 
3. 通过调用View.onDraw(canvas)绘制View本身的内容 
4. 通过 dispatchDraw(canvas)绘制自己的孩子,dispatchDraw->drawChild->child.draw(canvas) 这样的调用过程被用来保证每个子 View 的 draw 函数都被调用 
5. 如果需要，绘制淡入淡出相关的内容并恢复保存的画布所在的层（layer） 
6. 绘制修饰的内容（例如滚动条）

当一个 ChildView 要重画时，它会调用其成员函数 invalidate() 函数将通知其 ParentView 这个 ChildView 要重画，这个过程一直向上遍历到 ViewRoot，当 ViewRoot 收到这个通知后就会调用上面提到的 ViewRoot 中的 draw 函数从而完成绘制。Android 动画就是通过 ParentView 来不断调整 ChildView 的画布坐标系来实现的。 

