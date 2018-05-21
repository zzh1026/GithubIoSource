---
title: Android5-7重要新特性
comments: false
date: 2018-05-21 09:56:58
tags: Android新特性
categories: Android新特性

---

# 简介 #

随着Android系统的发展，新的版本会引入一些新的特性, 从Android5.0开始,Android引入了很多到现在为止都非常实用的新特性.这些新特性虽然也随着时间可能不断会更新, 但是大体上都是一脉相承的, 本篇文章对常用的 Android5.0 - Android8.0 版本的一些新特性进行梳理

<!-- more -->

# Android5.0新特性 #

![](https://upload-images.jianshu.io/upload_images/3449733-2ae2c888d6865772.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

## 新增加了 Design Library 设计风格	 ##

新增加的 Design Library 增加了一些控件 ,下面对一些常用的 5.0更新的控件做一下梳理, 不在 design 包下的会标出

主要使用 CoordinatorLayout 布局的

- CoordinatorLayout ,主要用来实现界面的交互效果, 它最重要的就是behavior.
- CollapsingToolbarLayout ,用于包含Toolbar和View或 ViewGroup, 提供滑动中的渐变和视差效果, 用于配合 CoordinatorLayout 做好看的效果
- AppBarLayout ,用AppBarLayout包裹的子view会以一个整体的形式作为AppBar，从而具备统一的风格 用于配合 CoordinatorLayout 做好看的效果
- ToolBar 用来替换actionBar (在 v7 支持包下)
- FloatingActionButton ,design下的重要悬浮界面效果的button 可以实现很多不错的效果  
- BottomNavigationView ,底部导航栏
- NavigationView ,侧滑菜单导航栏
- Snackbar ,显示提示, 类似于Toast功能,但是更加强大.

- TextInputLayout ,输入text 的layout
- CheckableImageButton ,可选中的image ,配合 TextInputLayout中的用来显示是否可见的图标
- TextInputEditText, 包裹在 TextInputLayout 中的输入框

- TabLayout ,类似于 viewpagerindercatiy , 主要作为viewpager的指示器来使用 ,但是这个控件的牛逼之处在于可以独立使用.

- CardView ,卡片view (不在 design包下,想要使用需要独立引用: implementation 'com.android.support:cardview-v7:24.1.1')
- RecyclerView ,用来替换 listview 和 gridview , 解耦更好, 更加清晰 (不在 design 包下,想要使用需要独立引用)
- NestedScrolling ,用于在 design 风格下代替 Scrollview ,很牛逼, 完美嵌套 多层 Recyclerview.(在v4支持包下)

## View的高度与阴影 ##
View新增属性z轴，用来体现Material Design中的层次，影响因素2个：elevation和translationZ(本身高度 和 向上偏移的高度)

View高度= elevation + translationZ 

elevation表示view的高度，高度越大，阴影越大，可以在xml中直接使用属性， 也可以在代码中使用

    view.setEvelvation()
    android:elevation="10dp"

transtionZ属性表示view在Z方向移动的距离，一般用于属性动画中;	

    android:translationZ="10dp"

高度影响View的绘制顺序，过去是按View添加顺序绘制，先添加的先绘制，现在高度小的先绘制，因为高度小的，层级低，在下面， 高度相同的，按添加顺序绘制

注意：

    - 如果View的背景色为透明，则不会显示出阴影效果 (这个很重要, 不设置背景的时候 背景默认为 透明,所以也不会显示阴影效果)
    - 只有子View的大小比父View小时，阴影才能显示出来

## View的轮廓与裁剪(在Android5.1以及以上才有效果) ##

View增加了轮廓概念,轮廓用来表示怎么显示阴影,也就是说轮廓什么形状，阴影就显示什么形状。

View的轮廓可以通过outlineProvider属性设置，默认是依据于background的，还有其他3个取值：bounds,none,paddingBounds	

	android:outlineProvider="bounds"

- none：即使设置了evaluation也不显示阴影
- background：按背景来显示轮廓，如果background是颜色值，则轮廓就是view的大小，如果是shape，则按shape指定的形状作为轮廓
- bounds: View的矩形大小作轮廓
- paddedBounds: View的矩形大小减去padding的值后的大小作轮廓。

也可以通过代码进行设置

    view.setOutlineProvider(new ViewOutlineProvider(){
    	public void getOutLine(view,outine){
    		outline.setOval(0,0,view.getWidth(),view.getHeight());
    	}
    });

## Palette ##

使用Palette可以让我们从一张图片中拾取颜色，将拾取到的颜色赋予ActionBar，StatusBar以及背景色可以让界面色调实现统一

    Palette p = Palette.generate(Bitmap bitmap);

    //或者
    Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener() {
        public void onGenerated(Palette palette) { }
    });

## 水波纹动画 ##

**Android5.0 可以 让我们自定义水波纹动画以及状态选择器动画**

首先，在Android5.0以上，点击效果默认自带水波纹效果，并且有2种选择：

    //矩形边框水波纹
	android:background="?android:attr/selectableItemBackground"

    //无边框限制水波纹
    android:background="?android:attr/selectableItemBackgroundBorderless"

>这两种方式都是系统自带的效果

也可以自定义水波纹动画

    //在指定view的指定位置，以startRadius为起始半径，endRadius为最终半径，绘制水波纹动画
    Animator anim = ViewAnimationUtils.createCircularReveal(view, centerX, centerY,startRadius,endRadius);
    anim.start();

使用ViewAnimationUtils创建圆形水波纹动画，注意该动画不能在Activity的onCreate方法中执行：

    Animator circularReveal = ViewAnimationUtils.createCircularReveal(text, 0, text.getHeight() , 1f, text.getWidth()*2);
    circularReveal.setDuration(1000);
    circularReveal.start();

或者  使用ripple标签或者RippleDrawable可以更改控件水波纹动画颜色

通过stateListAnimator属性指定状态选择器的动画：

    android:stateListAnimator="@drawable/selector_anim"

状态选择器文件中需要加入objectAnimator标签：

    android:duration = "@android:integer/configshortAnimTime"
    android:valueTo = "0.2"
    android:valueFrom = "1"
    android:valueType = "floatType"

## 自定义状态栏、标题栏、导航栏的颜色 ##

# 控件详解 #

## CoordinatorLayout ##

1. 可以协调多个布局间的位置关系。让FloatActionBar上下滑动，为Snackbar留出空间；拓展或折叠toolbar；控制view扩展或收缩，以及大小比例
2. CoordinatorLayout作为根布局使用
3. 配合FloatActionBar和SnackBar使用
    - 布局里添加FAB，当界面上显示Snackbar的时候会自动的偏移FAB的位置
4. 配合AppBarLayout和toolbar使用
    - 用AppBarLayout包裹的子view会以一个整体的形式作为AppBar，从而具备统一的风格
    - 设置toolbar属性可以使toolbar随着界面滑动而隐藏/显示
    
            app:layout_scrollFlags="scroll|enterAlways"
            // scroll 表示该view可以被折叠
			// enterAlways 表示向上滑动则隐藏ToolBar，向下滑动则显示
			// exitUntilCollapsed 将关闭滚动直到它被折叠起来(有 minHeight) 并且一直保持这样
			// enterAlwaysCollapsed 定义了 View 是如何回到屏幕的，当你的 view 已经声明了一个minHeight, 并且你使用了这个标志，你的 View 只有在回到这个最小的高度的时候才会展开，只有当 view 已经到达顶部之后它才会重新展开全部高度。

    - 可滚动的控件需要设置属性
        
            app:layout_behavior="@string/appbar_scrolling_view_behavior"
            // 标识自己发起的滚动可以导致AppBar收缩
>**appbar_scrolling_view_behavior这个String类型是固定的,代表与 appbar 和 scrollingview 的互动,这样 appbarlayout的 scrollFlags 才会生效, 其实 在 CoordinatorLayout 中, beheavior 的作用是相互的. 而 AppBarLayout 默认已经使用滑动的 scrolling view 的behavior**

    - 带layout_scrollFlags的view需要放在布局的前面，这样收回的view才能够正常退出，而固定的view继续留在顶部
    
5. 配合AppBarLayout和CollapsingToolbarLayout使用

    - AppBarLayout用于包裹且仅包裹CollapsingToolbarLayout，使得CollapsingToolbarLayout作为AppBar而存在
    - CollapsingToolbarLayout用于包含Toolbar和ImageView, 提供滑动中的渐变和视差效果
    - 设置CollapsingToolbarLayout属性
    
            app:expandedtitleMarginStart="10dp"// 指定文字和左边缘的间距
			app:contextScrim="?attr/colorPrimary"//折叠后容器的颜色
			app:layout_scrollFlag="scroll|exitUntilCollapsed"
			//拦截滚动的事件
			// enterAlwaysCollpsed 
			// exitUntilCollapsed 可以让ToolBar固定在最顶部，而不伴随手势的滚动隐藏
    - 设置ImageView属性

            app:layout_collapseMode="parallax"
            // parallax模式：在内容滚动时，CollapsingToolbarLayout里的view可以同时滚动，造出视差效果
    - 设置Toolbar属性
 
            app:layout_collapseMode="pin"
            // pin模式：当CollapsingToolbarLayout完全收缩后，继续保留在屏幕上
            android:layout_height="?attr/actionbarSize" // 设置高度为actionbar的高度

    - 手势滑动时，修改toolbar文字大小，文字颜色

        - 获取到CollapsingToolbarLayout对象
        - 设置标题
        
        	       collapsingToolbarLayout.setTitle
        - 设置展开状态的颜色
        
        	       collapsingToolbarLayout.ExpandedTitleColor
        - 设置折叠状态的颜色
        
                   collapsingToolbarLayout.setCollapsedTitleTextColor

## FloatingActionButton(FAB) ##

- 用于显示一个悬浮在界面上的按钮，可以设置点击事件(onClickListener)
- 设置按钮大小	app:fabSize="mini"
- 设置按钮背景色	app:backgroundTint="#fff"

## SnackBar ##

- 用于显示提示，官方建议用于替代Toast
- 对比Toast来使用
 
        Toast.makeText(context,msg,0).show();
        Snackbar.make(view,msg,0).show();
- 参数里View的作用 : 该view用于查找ParentView，以确定SnackBar的显示位置
- 设置点击事件 : snackBar.setAction("可点击的提示文字",onClickListener);
    

## ActionBarDrawerToggle ##

抽屉效果，可以关联toolbar ,效果是在界面划入, 但是不会对界面产生影响

- 创建DrawerToggle对象
    
        drawToggle = new ActionBarDrawerToggle(this,drawerLayout,toolbar,R.string.open,R.string.close);	
        //构造方法里关联了drawLayout和toolbar

- 设置监听

        drawerLayout.setDrawerListener(drawerToggle);

- 开启功能

        drawerToggle.sysncState();

## NavigationView ##

和 drawerToggle 的区别在于, 这个侧滑菜单会把右边的view顶向右边

- 用作侧滑菜单的侧边导航栏
- 必须嵌套在DrawerLayout里使用
    - DrawerLayout包含两个子布局，一个导航栏，另一个为显示的正文

- NavigationView的属性：
    - app:headerLayout，可选项，可以指定一个布局作为导航内容的Header
    - app:menu，必需项，指定一个menu，作为导航内容的菜单

- 导航栏的点击响应
    - navigationView.setNavigationItemSelectedListener
    - 该方法监听被点击的MenuItem，判断MenuItem对象处理对应的事件响应

## TextInputLayout ##

一个自带界面友好的错误提示输入框

TextInputLayout作为一个父容器控件，包装了新的EditText。通常，单独的EditText会在用户输入第一个字母之后隐藏hint提示信息,但是现在可以使用TextInputLayout来将EditText封装起来，提示信息会变成一个显示在EditText之上的floating label，这样用户就始终知道他们现在输入的是什么。同时，如果给EditText增加监听，还可以给它增加更多的floating label。

使用TextInputLayout包裹一个TextInputEditText

获取输入框:

    textInputLayout.getEditText(); 

显示错误提示
    
    textInputLayout.setErrorEnable(true);
    textInputLayout.setError(msg);

隐藏错误提示

    textInputLayout.setErrorEnable(false);

## TabLayout ##

- 方便的实现tab跟随Viewpager切换，不需要第三方或者自定义
- 在layout里使用tablayout
- 向tablayout对象addTab
    
        tabLayout.addTab(tabLayout.newTab().setText(""))
- 关联ViewPager

        方式一: viewpager.addOnPagerChangeListener(new TabLayoutOnPagerChangeListener(tabLayout))
        方式二：tabLayout.setupWithViewPager(viewPager)

- ViewPager使用的FragmentPagerAdapter :需要在getPagerTitle方法返回一个字符串，该字符串会作为对应position的tab的标题。
- 设置tab的点击事件 : tabLayout.setOnTabSelectedListener

## ToolBar ##

- 用于替代ActionBar，继承自ViewGroup可以任意包裹子布局，灵活性更高
- 设置主题，隐藏ActionBar : parent="Theme.AppCompat.NoActionBar"
- 如果界面继承自 AactionBarActivity，onCreate方法执行时设置ToolBar作为ActionBar

        setSupportActionBar(toolbar);

## RecycleView ##
很厉害 很常用,默认已经完全熟悉,不说了,如果不熟悉可以参考
[Android RecyclerView 使用完全解析 体验艺术般的控件](https://blog.csdn.net/lmj623565791/article/details/45059587)

## CardView ##
- 具备阴影的控件，也就是具备z轴海拔的控件
- 该View继承自FrameLayout，直接作为父布局包裹子控件即可

## SwipeRefreshLayout ##

v4包下的刷新控件

但是这个控件的功能比较单一, 但是设计思想非常厉害. 非侵入式可以说是非常好用了, 不过现在一般刷新都是使用的第三方的库,推荐

[SmartRefreshLayout](https://github.com/scwang90/SmartRefreshLayout)

# Android6.0 #

运行时权限处理更新, 超级大的版本更新.

# Android8.0 #

可查看 [Android8.0 新SupportLibrary26、27功能及变化介绍](https://juejin.im/post/5a3b9de2f265da43085e336b?utm_source=gold_browser_extension)

## XML中的字体 ##

字体可以作为一种新的资源类型

使用方式:

    1. 在res文件夹下创建font文件夹
    2. 将font类型文件（如dacing.ttf）拷贝到此目录下
    3. 或者将多个font文件创建成一个font族（font family）
    4. TextView 通过设置属性使用：android：fontFamily=“@font/lalala”
       Style 通过设置属性使用：<item name="android:fontFamily">@font/lobster</item>
       如果使用Support 库，需要使用对应的namespace
       在代码中获取字体资源：Typeface typeface = ResourcesCompat.getFont(context, R.font.myfont);

font family 举例

    <?xml version="1.0" encoding="utf-8"?>
    <font-family xmlns:android="http://schemas.android.com/apk/res/android">
    <font
         android:fontStyle="normal"
         android:fontWeight="400"
         android:font="@font/lobster_regular" />
    <font
         android:fontStyle="italic"
         android:fontWeight="400"
         android:font="@font/lobster_italic" />
    </font-family>

我们可以选择不将字体资源打包入apk，而是通过下载获得，这样做的好处有：

1. 减小apk体积
2. 提高应用安装成功率
3. 多个app可以共享一份相同的字体资源

原理：App通过fontContract向对应的FontProvider（如 google play）请求对应字体资源，如图

![](https://user-gold-cdn.xitu.io/2017/12/21/16078e0b7c1861f4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## TextView自动调整文字大小 ##

Support26库中提供了自动调整文字大小的TextView

> 注意：如果使用自动调整文字大小，不建议宽或者高设置为wrap_content，这会导致意想不到的错误

使用方式有三个方面（如果是support，注意使用namespace）：

- TextView默认是开启自动调整文字大小的

    我们可以在代码中通过setAutoSizeTextTypeWithDefaults(AUTO_SIZE_TEXT_TYPE_NONE)来关闭，这个属性也可以在xml文件中设置

    <?xml version="1.0" encoding="utf-8"?>
    <TextView
           android:layout_width="match_parent"
           android:layout_height="200dp"
           android:autoSizeTextType="uniform" />

- 通过最大最小值、粒度来控制自动调整大小的行为细节

    在代码中动态设置 setAutoSizeTextTypeUniformWithConfiguration(int autoSizeMinTextSize, int autoSizeMaxTextSize, int autoSizeStepGranularity, int unit)
    
    xml文件中设置 
    <TextView
       	 android:layout_width="match_parent"
       	 android:layout_height="200dp"
       	 android:autoSizeTextType="uniform"
       	 android:autoSizeMinTextSize="12sp"
       	 android:autoSizeMaxTextSize="100sp"
       	 android:autoSizeStepGranularity="2sp" />

- 我们也可以指定调整大小的具体值

    在代码中动态设置  setAutoSizeTextTypeUniformWithPresetSizes(int[] presetSizes, int unit) 
     
    xml文件中设置，首先在res/values/arrays.xmlres/values/arrays.xml中创建数组
    <resources>
         <array name="autosize_text_sizes">
       	    <item>10sp</item>
       	    <item>12sp</item>
       	    <item>20sp</item>
       	    <item>40sp</item>
       	    <item>100sp</item>
         </array>
    </resources>
    引用此数组
    <TextView
           android:layout_width="match_parent"
           android:layout_height="200dp"
           android:autoSizeTextType="uniform"
           android:autoSizePresetSizes="@array/autosize_text_sizes" />


## 基于物理的动画（DynamicAnimation) ##

- FlingAnimation（模拟摩擦，有一个初始速度，逐渐减慢至0）

        FlingAnimation(view,DynamicAnimation.TRANSLATION_Y).apply{
            setStartVelocity(5000f); //pixels per second
            friction = 1.5f; //摩擦系数
            start();
        }

- SpringAnimation(模拟弹簧,可以设置阻尼比，刚度，最后停下的位置等)

        SpringAnimation(view,SpringANimation.TRANSLATION.Y,0f).apply{
           spring.apply{
             dampingRatio = SpringForce.DAMPING>RATIO>LOW>BOUNCY;
             stiffness = SpringForce.STIFFNESS_LOW;
             finalPosition = 0f;
           }
            setStartVelocity(20000f);
            start();
        }

![](https://user-gold-cdn.xitu.io/2017/12/21/16078e0b5f1541b9?imageslim)
