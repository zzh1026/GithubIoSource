
---

title: Android软键盘
date: 2018-05-16 13:54:08
tags: Android,软键盘
categories: 
    - Android
    - 知识总结

---

## 1,WindowSoftInputMode属性 ##

活动的主窗口如何与包含屏幕上的软键盘窗口交互,这个属性的设置将会影响两件事情:

1,软键盘的状态:当活动(Activity)成为用户关注的焦点时，它是否隐藏或显示。

2,对活动主窗口进行的调整：无论是调整大小以便为软键盘腾出空间，还是在软键盘覆盖窗口的一部分，以便当前焦点内容可见。

<!-- more -->

### 1,属性详解 ###

>The setting must be one of the values listed in the following table, or a combination of one "state..." value plus one "adjust..." value. Setting multiple values in either group — multiple "state..." values, for example — has undefined results. Individual values are separated by a vertical bar (|). For example:

该属性可选的值有两部分，一部分为软键盘的状态控制，控制软键盘是隐藏还是显示，另一部分是Activity窗口的调整，以便腾出空间展示软键盘。        android:windowSoftInputMode的属性设置必须是下面中的一个值，或一个”state”值加一个”adjust”值的组合，各个值之间用 | 分开。

><activity android:windowSoftInputMode="stateVisible|adjustResize" . . . >

设置的值为1个或者 一个 state... 加 一个 adjust...

>Values set here (other than "stateUnspecified" and "adjustUnspecified") override values set in the theme.

此处设置的值（“stateUnspecified”和“adjustUnspecified”除外）将覆盖主题中设置的值。
PS:这个是很好理解的,  正如在代码中设置会覆盖activity注册声明中的值, activity注册声明中的值也会覆盖主题中设置的值:  代码设置 > 注册声明 >　主题设置

用来设置窗口软键盘的交互模式，其属性一共有9个取值

## state状态的:6种 ##

#### stateUnspecified　－－－－－　SOFT_INPUT_STATE_UNSPECIFIED ####
>no state has been specified	(未指定状态)

默认交互方式，系统会根据界面采取相应的软键盘的显示模式。比如，当界面上只有输入框和按钮的时候，软键盘就不会自动弹出，但是当有获得焦点的输入框的界面有滚动的需求的时候，会自动弹出软键盘，例如外层为ScrollView。阻止键盘弹出的一个解决方案就是，在xml文件中，设置一个非输入框控件获取焦点。

#### stateUnchanged 	-----------  SOFT_INPUT_STATE_UNCHANGED ####
>please don't change the state of the soft input area.(不改变软键盘的状态,当前的键盘是否显隐,取决于上一个界面的软键盘状态,无论是隐藏还是显示)

保持当前软键盘状态不变。举个例子，假如上个界面键盘是隐藏的，那么跳转到该界面之后，软键盘也是隐藏的；如果上个界面是显示的，那么跳转该界面后，软键盘也是显示状态。

#### stateHidden ---------------	 SOFT_INPUT_STATE_HIDDEN ####
>please hide any soft input area when normally appropriate (when the user is navigating forward to your window).(在普通适当的情况隐藏软键盘,当用户正在该界面操作浏览的时候)

当设置该状态时，软键盘总是被隐藏，不管是否有输入的需求。

#### stateAlwaysHidden ---------  SOFT_INPUT_STATE_ALWAYS_HIDDEN ####
>please always hide any soft input area when this window receives focus.(当窗口获取到焦点的时候也保持隐藏)

当设置该状态时，软键盘总是被隐藏，和stateHidden不同的是，当我们跳转到下个界面，如果下个页面的软键盘是显示的，而我们再次回来的时候，软键盘就会隐藏起来。

#### stateVisible --------------  SOFT_INPUT_STATE_VISIBLE ####
>please show the soft input area when normally appropriate (when the user is navigating forward to your window).(在普通适当的情况展示软键盘,当用户正在该界面操作浏览的时候)

当设置为这个状态时，软键盘总是可见的，即使在界面上没有输入框的情况下也可以强制弹出来出来。。

#### stateAlwaysVisible --------  SOFT_INPUT_STATE_ALWAYS_VISIBLE ####
>please always make the soft input area visible when this window receives input focus.(当这个窗口接收输入焦点保持可见)

## adjust状态的:4种 ##

#### adjustUnspecified ---------  SOFT_INPUT_ADJUST_UNSPECIFIED ####
>nothing specified. The system will try to pick one or the other depending on the contents of the window.(默认,)

设置软键盘与软件的显示内容之间的显示关系，默认的设置模式。在这中情况下，系统会根据界面选择不同的模式。如果界面里面有可以滚动的控件，比如ScrowView，系统会减小可以滚动的界面的大小，从而保证即使软键盘显示出来了，也能够看到所有的内容。如果布局里面没有滚动的控件，那么软键盘可能就会盖住一些内容。没有滚动控件，软键盘下面的布局都被遮挡住，若想修改，只能隐藏软键盘，然后选择。而且，重点注意一下上面的布局，当我们选择的输入框偏下的时候，上面的标题栏和布局被软键盘顶上去了。

#### adjustResize --------------  SOFT_INPUT_ADJUST_RESIZE ####

这个属性表示Activity的主窗口总是会被调整大小，从而保证软键盘显示空间。

该模式下窗口总是调整屏幕的大小用以保证软键盘的显示空间；这个选项不能和adjustPan同时使用，如果这两个属性都没有被设置，系统会根据窗口中的布局自动选择其中一个。

#### adjustPan -----------------  SOFT_INPUT_ADJUST_PAN ####

如果设置为这个属性，那么Activity的屏幕大小并不会调整来保证软键盘的空间，而是采取了另外一种策略，系统会通过布局的移动，来保证用户要进行输入的输入框肯定在用户的视野范围内，从而让用户可以看到自己输入的内容。对于没有滚动控件的布局来说，这个其实就是默认的设置，如果我们选择的位置偏下，上面的标题栏和部分控件会被顶上去。

对于没有滚动控件的布局来说，adjustPan就是默认的设置。
 
####adjustNothing -------------  SOFT_INPUT_ADJUST_NOTHING ####

#### 备注：如果我们不设置"adjust..."的属性，对于没有滚动控件的布局来说，采用的是adjustPan方式，而对于有滚动控件的布局，则是采用的adjustResize方式。 ####

#### 备注2: 对于没有滚动控件的布局来说，整个布局会往上偏移（包括标题等）以保证输入框的可见。如果有滚动控件，那么就是内容网上偏移，标题不会动，还可以通关滚动来查看被顶上去的内容，而这点是不可滚动的布局所不具备的 ####

#### 备注3: 根据这一原理，我们就可以把开发中遇到的软键盘遮挡页面的问题，利用ScrollView当做根布局，让系统采用adjustResize模式，很好地解决这一问题。 ####

### 2,使用方式 ###

1,代码实现方式：

	//在activity中的setContentView之前写上以下代码
	getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);

2,xml实现方式

	//在 项目的AndroidManifest.xml文件中界面对应的<activity>里加入
	android:windowSoftInputMode="adjustPan"

## 2,手动打开,关闭软键盘 ##
[关于键盘的自动弹出和不自动弹出可以看这里](http://www.jianshu.com/p/50c35e1bf09b)

在开发过程中，难免会遇到想手动打开或者关闭软键盘的情况，这时使用以下代码不失为一种好办法。

	/**
	 * 动态显示隐藏软键盘,如果显示则隐藏,如果隐藏则显示
	 *
	 * @param context context
	 */
	public static void toggleSoftInput(Context context) {
	    InputMethodManager imm = (InputMethodManager) context.getSystemService(Context.INPUT_METHOD_SERVICE);
	    imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS);
	}

#### 打开软键盘 ####
	//这个不一定起作用
	public static void openSoftInput(Context context) {
        InputMethodManager imm = (InputMethodManager) context.getSystemService(Context.INPUT_METHOD_SERVICE);
        imm.toggleSoftInput(0, InputMethodManager.HIDE_IMPLICIT_ONLY);
    }

	或者

	InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
	imm.showSoftInput(view,InputMethodManager.SHOW_FORCED);	

	或者

	//使用EditText来弹出软键盘
	InputMethodManager inputManager =(InputMethodManager)etInput.getContext ().getSystemService(Context.INPUT_METHOD_SERVICE);
    inputManager.showSoftInput(etInput, 0);

#### 隐藏软键盘 ####

	InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
	imm.hideSoftInputFromWindow(view.getWindowToken(), 0)

	或者

	int flags = WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM; 
    getWindow().addFlags(flags);

	或者

	//在onclick事件下.以下方法可行.(如果是EditText失去焦点/得到焦点,没有效果)
    InputMethodManager im = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE); 
    im.hideSoftInputFromWindow(getCurrentFocus().getApplicationWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);

#### 获取输入法打开的状态	 ####

	InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
	boolean isOpen=imm.isActive();//isOpen若返回true，则表示输入法打开
PS：输入法打开的意思并不是说软键盘弹出，这个两个概念

## 3,软键盘的Enter键 ##

### 1,使用方式 ###

1,当layout中有多个EditText，把每个控件的android:singleLine的属性都被设置成true的情况下，软键盘的Enter键上 的文字会变成“Next”，按下后下个EditText会自动获得焦点（实现了“Next”的功能）；当最后一个控件获得焦点的时候，Enter键上的文 字会变成“Done”，按下后软键盘会自动隐藏起来。

2,把EditText的ImeOptions属性设置成不同的值，Enter键上可以显示不同的文字或图案

	actionNone : 回车键，按下后光标到下一行
	actionGo ： Go，
	actionSearch ： 一个放大镜
	actionSend ： Send
	actionNext ： Next
	actionDone ： Done，隐藏软键盘，即使不是最后一个文本输入框

	inputView.setImeOptions(EditorInfo.IME_ACTION_SEARCH);
	editText.setOnEditorActionListener(new TextView.OnEditorActionListener() {
	            public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
	                if (actionId == EditorInfo.IME_ACTION_SEARCH || (event != null && event.getKeyCode() == KeyEvent.KEYCODE_ENTER)) {
	                     //do something;              
	                    return true;
	                }
	                return false;
	            }
	        });

# 部分坑 #

### 1,软键盘无法顶起页面 ###

开发中有个需求是将页面底部的一个按钮顶起，但是开发时发现Android5.0以后的版本设置了adjustResize属性后无法成功顶起。纠结了好久，最后在stackoverflow找到解决方案，那就是在根布局上加上fitsSystemWindow=”true”即可。

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"   	android:layout_width="match_parent" 
		android:layout_height="match_parent" 	
		android:fitsSystemWindows="true">

这里的fitsSystemWindow具体的作用就是你的contentview是否忽略actionbar,title,屏幕的底部虚拟按键，将整个屏幕当作可用的空间。 

正常情况，contentview可用的空间是去除了actionbar,title,底部按键的空间后剩余的可用区域；这个属性设置为true,则忽略，false则不忽略

### 2,自定义软键盘按钮功能无效 ###

在edittext上加入Android:imeOptions=”actionSearch”这个属性没响应，最后发现在2.3及以上版本不起作用，解决方案：加上

 	android:singleLine="true"

因为输入法键盘右下角默认的回车键本来就是换行用的，当设置单行后，回车换行就失去作用了，这样就可以设置为搜索、发送、go等等。


# Android键盘控制器InputMethodManager #

### 获取InputMethodManager对象： ###

	InputMethodManager imm = (InputMethodManager) getSystemService(Service.INPUT_METHOD_SERVICE);

### 切换打开/隐藏状态 ###

	imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS);

### 打开键盘 ###

	imm.showSoftInput(view,InputMethodManager.SHOW_FORCED);

### 隐藏键盘 ###

	imm.hideSoftInputFromWindow(view.getWindowToken(), 0);

### 获取输入法状态 ###

	boolean isOpen=imm.isActive();



