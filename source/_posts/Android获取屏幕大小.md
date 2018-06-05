---
title: Android获取屏幕大小汇总
comments: false
date: 2018-05-18 16:53:01
tags:   Android屏幕
categories:
    - Android
    - 知识总结
    
---

## 简介 ##

以前,每次涉及到要获取屏幕宽高的时候总是比较纠结, 因为感觉获取屏幕大小的方法有点乱,方法有点多.不容易记住,虽然一直有之前的工具方法, 但是这个方法随着时间的推移是否还准确等等问题都比较纠结, 总之, 是时候总结一波了, 不把所有的方法都搞搞怎么知道那个好用呢?

<!-- more -->

## update ##
本篇blog 是大约1年前的blog ,最近发现并未过时, 为了在个人blog上记录 的同时还能再加深一下了解和记忆, 所以重新对该blog进行了梳理.

## Android获取屏幕大小方法汇总 ##

### 通过 Service 获取 Display 直接调用方法 ###

    WindowManager wm = (WindowManager) this.getSystemService(Context.WINDOW_SERVICE);
    Display defaultDisplay = wm.getDefaultDisplay();
    int width = defaultDisplay.getWidth();		//getWidth() 和 getHeight() 都过时了,所以没必要用了
    int height = defaultDisplay.getHeight();

![](http://zhaozehui.cn/images/blogiamges/image_6.png)

总结:  通过获取service的方法获取到 window_service 来得到windowmanger, 然后通过getDefaultDisplay方法得到 display类, 再获取宽高, 在实际过程中, activity对 getSystemService 进行了重写, 如果是 WINDOW_SERVICE 则会直接返回 WindowManager 的引用, 使用context也可以获取到该引用.

其过程为: 

    class Display
		public int getWidth() {
	        synchronized (this) {
	            updateCachedAppSizeIfNeededLocked();	//先执行这个方法(进入该方法)
	            return mCachedAppWidthCompat;	//然后返回这个值
	        }
	    }

	    class Display
	    private void updateCachedAppSizeIfNeededLocked() {
	        long now = SystemClock.uptimeMillis();
	        if (now > mLastCachedAppSizeUpdate + CACHED_APP_SIZE_DURATION_MILLIS) {
	            updateDisplayInfoLocked();	//多次执行这个方法
	            mDisplayInfo.getAppMetrics(mTempMetrics, mDisplayAdjustments);		//通过该方法获取(进入该方法)
	            mCachedAppWidthCompat = mTempMetrics.widthPixels;	//获取到的宽高
	            mCachedAppHeightCompat = mTempMetrics.heightPixels;
	            mLastCachedAppSizeUpdate = now;
	        }
	    }

	    class  DisplayInfo
	    public void getAppMetrics(DisplayMetrics outMetrics, DisplayAdjustments displayAdjustments) {
	        getMetricsWithSize(outMetrics, displayAdjustments.getCompatibilityInfo(),		//通过这个方法获取到大小
	                displayAdjustments.getConfiguration(), appWidth, appHeight);
	    }

### 通过 Service 获取 Display 使用Point  ###

    WindowManager wm = (WindowManager) this.getSystemService(Context.WINDOW_SERVICE);	//先获通过服务获取到windowmanager
    Display defaultDisplay = wm.getDefaultDisplay();		//其次获取到display对象
    int a = 1;
    int b = 2;
    Point point = new Point(a, b);						
    wm.getDefaultDisplay().getSize(point);			//获取大小

>
 
       class Display
       public void getSize(Point outSize) {
    	    synchronized (this) {
    	        updateDisplayInfoLocked();		//先执行这个方法(和上面一样,就不进入该方法了)
    	        mDisplayInfo.getAppMetrics(mTempMetrics, mDisplayAdjustments);	//这个和上面也一样也调用到了getAppmetrics
    	        outSize.x = mTempMetrics.widthPixels;
    	        outSize.y = mTempMetrics.heightPixels;
    	    }

### 通过直接获取 WindowManager ###

    WindowManager wm1 = this.getWindowManager();			//方法三和方法1是一样的,区别仅仅是获取windowmanager的方式不一样,也就意味着也需要调用getAppmetrics来进行测量
    Display defaultDisplay1 = wm1.getDefaultDisplay();
    int width1 = defaultDisplay1.getWidth();
    int height1 = defaultDisplay1.getHeight();
   

>这个方法是Activity中的方法,即 使用Context是无法使用该方法的

    WindowManager manager = this.getWindowManager();		//总之就是先获取到windowmanger ,前4个的开头本质上是一样的
    DisplayMetrics outMetrics = new DisplayMetrics();		
    manager.getDefaultDisplay().getMetrics(outMetrics);		//获取到display对象调用getmetris方法, 和传入point的方法去point.x, point.y是一样的(进入该方法)
    int width2 = outMetrics.widthPixels;
    int height2 = outMetrics.heightPixels;

### 通过Resources获取 ###

    Resources resources = this.getResources();			//获取resources资源文件类
    DisplayMetrics dm = resources.getDisplayMetrics();	//同样的获取一个这个,应该和4是一样的通过windowmanage获取到display再通过getMetrics来获取(进入该方法)
    int width3 = dm.widthPixels;
    int height3 = dm.heightPixels;

部分源码

        class  resources
    	    public DisplayMetrics getDisplayMetrics() {
    	        return mResourcesImpl.getDisplayMetrics();		(继续进入该方法)
    	    }

		class  mResourcesImpl
    		DisplayMetrics getDisplayMetrics() {		
    	        return mMetrics;		//这个东西是 private final DisplayMetrics mMetrics = new DisplayMetrics(); ,在该类创建的时候就创建了
    	    }

	    class  ResourcesImpl
    		public ResourcesImpl(@NonNull AssetManager assets, @Nullable DisplayMetrics metrics,
    	            @Nullable Configuration config, @NonNull DisplayAdjustments displayAdjustments) {
    	        mAssets = assets;
    	        mMetrics.setToDefaults();			//初始化这个
    	        mDisplayAdjustments = displayAdjustments;
    	        updateConfiguration(config, metrics, displayAdjustments.getCompatibilityInfo());	//初始化这个
    	        mAssets.ensureStringBlocks();
    	    }

 		class  ResourcesImpl
    		public void setToDefaults() {	//初始化
    	        widthPixels = 0;
    	        heightPixels = 0;
    	        density =  DENSITY_DEVICE / (float) DENSITY_DEFAULT;
    	        densityDpi =  DENSITY_DEVICE;
    	        scaledDensity = density;
    	        xdpi = DENSITY_DEVICE;
    	        ydpi = DENSITY_DEVICE;
    	        noncompatWidthPixels = widthPixels;
    	        noncompatHeightPixels = heightPixels;
    	        noncompatDensity = density;
    	        noncompatDensityDpi = densityDpi;
    	        noncompatScaledDensity = scaledDensity;
    	        noncompatXdpi = xdpi;
    	        noncompatYdpi = ydpi;
    	    }


## 结论 ##
归根揭底是有两种获取屏幕信息的方法 

- 通过windowmanager的getDefaultDisplay方法得到Display对象,然后通过getWidth(),getSize(Point point),getMetrics(DisplayMetrics displayMetrics)来获取屏幕宽高信息,但是 这些方法的内部都是一个道理 , 即 其内部会通过 DisplayInfo 对象来调用 getAppMetrics(DisplayMetrics outMetrics, DisplayAdjustments displayAdjustments) 来获取信息,仅此而已.
- 通过getResources来获取到resources对象 ,然后调用getDisplayMetrics() 方法来得到 DisplayMetrics 的对象,该对象即包含了屏幕的宽高信息

Context.getResources().getDisplayMetrics()依赖于手机系统，获取到的是系统的屏幕信息

WindowManager.getDefaultDisplay().getMetrics(dm)是获取到Activity的实际屏幕信息
	
1.	 通过Display获取	
	
        Display defaultDisplay = getWindowManager().getDefaultDisplay();

    	DisplayMetrics dm = new DisplayMetrics();
    
    	defaultDisplay.getMetrics(dm);     //将屏幕的信息保存在DisplayMetrics中,是实际屏幕信息
        defaultDisplay.getRealMetrics(dm);  //将屏幕的信息保存在DisplayMetrics中,是系统的屏幕信息

        Point point = new Point();
        
        defaultDisplay.getSize(point);    //将屏幕的信息保存在point中,源码中是创建了一个 DisplayMetrics ,然后将 DisplayMetrics的值赋值给 Point , 因此他们是一致的;
        defaultDisplay.getRealSize(point);  //将屏幕的信息保存在point中

        Rect rect = new Rect();
        defaultDisplay.getRectSize(rect); //将屏幕的信息保存在Rect中,源码中是创建了一个 DisplayMetrics ,然后将 DisplayMetrics的值赋值给 Rect 的 right 和 bottom 中 , 因此他们本质上是一致的;

因此 , 凡是通过 Display 获取的屏幕信息 实际上都是通过 Display 中的 getMetrics 或 getRealMetrics 方法 . 区别为 getMetrics 获取的是实际屏幕信息, 而 getRealMetrics 获取的是系统的屏幕信息

2. 通过Resources获取

    DisplayMetrics dm2 = getResources().getDisplayMetrics();
    将屏幕的信息保存在返回的DisplayMetrics中 是实际屏幕信息

## 最后 ##

获取屏幕的信息比较方便的方法就是 获取到 Display ,一般的在Activity中可以方便的获取到其 WindowManager 的引用 而 WindowManager实际用到的方法并不多

![](http://zhaozehui.cn/images/blogiamges/image_7.png)

同时 getSize 和  getRectSize 等方法实际上都是 通过 getMetrics 的方法获取到的, 因此在获取的时候最好通过 getMetrics(DisplayMetrics outMetrics) 方法来进行处理. 

或者是通过Resources 中的 DisplayMetrics dm2 = getResources().getDisplayMetrics(); 来获取, 因为 DisplayMetrics类封装了屏幕的很多信息

![](http://zhaozehui.cn/images/blogiamges/image_8.png)

最后上一份当年实测代码和数据, 可以方便的分清楚实际屏幕信息和系统的屏幕信息的区别

        WindowManager manager = this.getWindowManager();

        DisplayMetrics dm1 = new DisplayMetrics();
        manager.getDefaultDisplay().getMetrics(dm1);
        int width1 = dm1.widthPixels;
        int height1 = dm1.heightPixels;
        ALog.i("第一种方法的 宽 :" + width1 + " ,高 :" + height1 + " ,xdip="
                + dm1.xdpi + " , ydip=" + dm1.ydpi);

        DisplayMetrics dm2 = new DisplayMetrics();
        manager.getDefaultDisplay().getRealMetrics(dm2);
        int width2 = dm2.widthPixels;
        int height2 = dm2.heightPixels;

        ALog.i("第二种方法的real 宽 :" + width2 + " ,高 :"
                + height2 + " ,xdip=" + dm2.xdpi + " , ydip=" + dm2.ydpi);

        Resources resources = this.getResources();
        DisplayMetrics dm3 = resources.getDisplayMetrics();
        int width3 = dm3.widthPixels;
        int height3 = dm3.heightPixels;
        ALog.i("第三种方法的 宽 :" + width3 + " ,高 :"
                + height3 + " ,xdip=" + dm3.xdpi + " , ydip=" + dm3.ydpi);


#### mi5sp ####

    第一种方法的 宽 :1080 ,高 :1920 ,xdip=386.366 , ydip=387.047
    第二种方法的real 宽 :1080 ,高 :1920 ,xdip=386.366 , ydip=387.047
    第三种方法的 宽 :1080 ,高 :1920 ,xdip=386.366 , ydip=387.047

#### mi4c ####

    第一种方法的 宽 :1080 ,高 :1920 ,xdip=449.704 , ydip=447.412
	第二种方法的real 宽 :1080 ,高 :1920 ,xdip=449.704 , ydip=447.412
	第三种方法的 宽 :1080 ,高 :1920 ,xdip=449.704 , ydip=447.412

#### 华为p9 ####

    第一种方法的 宽 :1080 ,高 :1794 ,xdip=428.625 , ydip=427.789
    第二种方法的real 宽 :1080 ,高 :1920 ,xdip=428.625 , ydip=427.789
    第三种方法的 宽 :1080 ,高 :1794 ,xdip=428.625 , ydip=427.789

#### nexus5 ####

    第一种方法的 宽 :1080 ,高 :1794 ,xdip=422.03 , ydip=424.069
    第二种方法的real 宽 :1080 ,高 :1920 ,xdip=422.03 , ydip=424.069
    第三种方法的 宽 :1080 ,高 :1794 ,xdip=422.03 , ydip=424.069

#### 华为p8 ####

    第一种方法的 宽 :1440 ,高 :2408 ,xdip=487.68 , ydip=488.902
	第二种方法的real 宽 :1440 ,高 :2560 ,xdip=487.68 , ydip=488.902
	第三种方法的 宽 :1440 ,高 :2408 ,xdip=487.68 , ydip=488.902



    