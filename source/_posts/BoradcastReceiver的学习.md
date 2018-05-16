---

title: BoradcastReceiver的使用
date: 2018-05-16 13:54:08
tags: Android,Boradcast,Receiver
categories: 学习知识

---

BroadcastReceiver翻译为广播接收者，Broadcast是一种广泛运用在应用程序之间的传输信息的机制，简单的可以理解为传统意义上的电台广播，通俗一点，发布失物招领。

广播机制是一个典型的发布—订阅模式，也就是我们所说的观察者模式。广播最大的特点就是发送方并不关心接收方是否接到数据，也不关心接收方是如何处理数据的，通过这样的形式来达到接、收双方的完全解耦合

<!-- more -->

## 1,普通广播 ##

普通广播是完全异步的，通过Context的sendBroadcast()方法来发送，消息传递效率比较高，但所有receivers（接收器）的执行顺序不确定。

缺点是：接收者不能将处理结果传递给下一个接收者，并且无法终止广播Intent的传播，直到没有与之匹配的广播接收器为止。

### 自定义广播 ###

#### 1,创建广播 ####

继承BroadcastReceiver并实现onReceive()方法

	public class SendMyBroadCast extends BroadcastReceiver {
	
	    @Override
	    public void onReceive(Context context, Intent intent) {
	
	    }
	}

#### 2,注册广播 ####

BroadcastReceiver是四大组件之一，所以必须要注册,其注册方法有两种：

1,在 androidmanifests 中进行配置

	<receiver android:name=".broadcast.SendMyBroadCast" >
         <intent-filter>
             <action android:name="con.neishenme.sendmyself" />
         </intent-filter>
    </receiver>

	这里需要加入intent-filter的action中的name属性，表示我们监听的内容。当有广播发送时，需要判断该广播是否和我们监听的内容一致，如果一致则接收.

2,通过代码动态配置

	SendMyBroadCast sendMyBroadCast = new SendMyBroadCast();
	IntentFilter filter = new IntentFilter("con.neishenme.sendmyself");
	registerReceiver(sendMyBroadCast, filter);

#### 3,取消广播注册 ####
BroadcastReceiver必须遵循生到死的周期，如果你是使用动态注册广播的则需要在Activity的onDestroy的时候取消注册; 如果是静态注册(在manifests中注册)则不需要取消注册.

	@Override
	protected void onDestroy() {
	    unregisterReceiver(sendMyBroadCast);
	    super.onDestroy();
	}

#### 4,发送广播 ####

	Intent intent = new Intent("con.neishenme.sendmyself");
	sendBroadcast(intent);

## 2,有序广播 ##

有序广播通过Context.sendOrderedBroadcast()来发送，所有的广播接收器优先级依次执行，广播接收器的优先级通过receiver的intent-filter中的android:priority属性来设置，数值越大优先级越高。

当广播接收器接收到广播后，可以使用setResult()函数来结果传给下一个广播接收器接收，然后通过getResult()函数来取得上个广播接收器接收返回的结果。

当广播接收器接收到广播后，也可以用abortBroadcast()函数来让系统拦截下来该广播，并将该广播丢弃，使该广播不再传送到别的广播接收器接收

### 有序广播的过程 ###

#### 1,创建广播 ####

	public class SendOrderMyBroadCast {
	    public static class HignPriority extends BroadcastReceiver {
	
	        @Override
	        public void onReceive(Context context, Intent intent) {
	            Log.i("hehe", "高权限获接收到了广播");
	
	            int resultCode = getResultCode();
	            Log.i("hehe", "高权限code为: " + resultCode);
	            String resultData = getResultData();
	            Log.i("hehe", "高权限resultData为: " + resultData);
	            Bundle resultExtras = getResultExtras(true);
	            Log.i("hehe", "高权限resultExtras为: " + resultExtras);
	
	            int code = 10;
	            String data = "hellohigh";
	            setResult(code, data, null);
	        }
	    }
	
	    public static class MidPriority extends BroadcastReceiver {
	
	        @Override
	        public void onReceive(Context context, Intent intent) {
	            Log.i("hehe", "中等权限获接收到了广播");
	
	            int resultCode = getResultCode();
	            Log.i("hehe", "中等权限code为: " + resultCode);
	            String resultData = getResultData();
	            Log.i("hehe", "中等权限resultData为: " + resultData);
	            Bundle resultExtras = getResultExtras(true);
	            Log.i("hehe", "中等权限resultExtras为: " + resultExtras);
	
	            int code = 100;
	            String data = "hellomid";
	            setResult(code, data, null);
	        }
	    }
	
	    public static class LowPriority extends BroadcastReceiver {
	
	        @Override
	        public void onReceive(Context context, Intent intent) {
	            Log.i("hehe", "低权限获接收到了广播");
	
	            int resultCode = getResultCode();
	            Log.i("hehe", "低权限code为: " + resultCode);
	            String resultData = getResultData();
	            Log.i("hehe", "低权限resultData为: " + resultData);
	            Bundle resultExtras = getResultExtras(true);
	            Log.i("hehe", "低权限resultExtras为: " + resultExtras);
	        }
	    }
	}

>要注意的是：内部类的BroadcastReceiver必须由public static修饰，否则会报错

#### 2,注册广播 ####
这里的注册方式和普通广播是一样的，这里的区别在于priority属性，确定了他们之间的优先级
这里也可以通过代码注册, 但是需要通过设置优先级

	<receiver android:name=".broadcast.SendOrderMyBroadCast$HignPriority">
	    <intent-filter android:priority="1000">
	        <action android:name="con.neishenme.sendmyself" />
	    </intent-filter>
	</receiver>

	<receiver android:name=".broadcast.SendOrderMyBroadCast$MidPriority">
	    <intent-filter android:priority="500">
	        <action android:name="con.neishenme.sendmyself" />
	    </intent-filter>
	</receiver>

	<receiver android:name=".broadcast.SendOrderMyBroadCast$LowPriority">
	    <intent-filter android:priority="100">
	        <action android:name="con.neishenme.sendmyself" />
	    </intent-filter>
	</receiver>

#### 3,发送广播 ####
和之前的不一样的地方，这里是使用sendOrderedBroadcast()发送有序广播

	public void send(View view) {
	    Intent intent = new Intent("con.neishenme.sendmyself");
	    sendOrderedBroadcast(intent, null);
	}

结果为:

	02-27 14:38:56.103 22772-22772/com.demo.downrefresh I/hehe: 高权限获接收到了广播
	02-27 14:38:56.103 22772-22772/com.demo.downrefresh I/hehe: 高权限code为: -1
	02-27 14:38:56.103 22772-22772/com.demo.downrefresh I/hehe: 高权限resultData为: null
	02-27 14:38:56.108 22772-22772/com.demo.downrefresh I/hehe: 中等权限获接收到了广播
	02-27 14:38:56.108 22772-22772/com.demo.downrefresh I/hehe: 中等权限code为: 10
	02-27 14:38:56.108 22772-22772/com.demo.downrefresh I/hehe: 中等权限resultData为: hellohigh
	02-27 14:38:56.111 22772-22772/com.demo.downrefresh I/hehe: 低权限获接收到了广播
	02-27 14:38:56.111 22772-22772/com.demo.downrefresh I/hehe: 低权限code为: 100
	02-27 14:38:56.111 22772-22772/com.demo.downrefresh I/hehe: 低权限resultData为: hellomid

要注意的是：

这里需要发送的是有序广播，否则在接收者中通过setResult()和getResult()方法会取不到值，因为只有有序广播才能传递结果

## 3,拦截广播 ##

通过在BroadcastReceiver中，执行abortBroadcast()方法，广播就不会继续往下传递了

在 onRecive 中调用该方法

	public static class HignPriority extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {
            Log.i("hehe", "高权限获接收到了广播");

            int resultCode = getResultCode();
            Log.i("hehe", "高权限code为: " + resultCode);
            String resultData = getResultData();
            Log.i("hehe", "高权限resultData为: " + resultData);

            int code = 10;
            String data = "hellohigh";
            setResult(code, data, null);

			//拦截广播
            abortBroadcast();
        }
    }

### 终结广播 ###

现在有这样的一个应用场景，按照上面的程序走，只能在第一个广播中被拦截住了，后面的广播则不执行。如果这个时候我们需要一个不管有没有被拦截都必须执行的广播，我们称为终结广播，那应该怎么办。

同样的，发送有序广播也考虑到这一点，通过以下代码来发送广播，并指定我们不管有没有被拦截都必须执行的终结广播

	Intent intent = new Intent("con.neishenme.sendmyself");
	//sendOrderedBroadcast(intent, null);
	sendOrderedBroadcast(intent, null, new SendOrderMyBroadCast.LowPriority(), new Handler(), 0, "hehe", null);

可以发现，之前只是有High的Log信息，因为是被拦截了，而Log信息多了一条Low，说明我们拦截后，还要执行终结广播。

#### 有两点需要注意 ####
#### 1,终结广播是肯定能获取到的,但是其获取getResultCode和getResultDate的时候是拦截的时候的值,abortBroadcast() 方法不分调用的顺序,在setResult之前和之后调用的情况是一样的.  PS: sendOrderedBroadcast有两个重载的方法 , 第二个方法代表  肯定可以执行到的广播 和 code,data和bundle的初始值 , 在第一个拦截后其设置的值 即为修改之后的值 ####

#### 2,在有序广播中,广播接受者必须是一个一个的收到广播的, 两个广播的priority值相同,那么接收广播的顺序是按照 在 manifest 声明顺序 或者 代码注册的时候 的注册顺序来决定的, 所以两个同样为 500的优先级的先注册的broadcast 可以通过 abortBroadcast() 方法来让后注册接收者 获取不到,即比较方法有两种, 先是通过 priority 的优先值 ,其次 priority相同通过注册的先后顺序来 比较 .####

## 4,本地广播 ##

在API21的Support v4包中新增本地广播，也就是LocalBroadcastManager。

由于之前的广播都是全局的，所有应用程序都可以接收到，这样就会带来安全隐患，所以我们使用LocalBroadcastManager只发送给自己应用内的信息广播，限制在进程内使用。

它的用法很简单，只需要把调用context的sendBroadcast、registerReceiver、unregisterReceiver的地方换为LocalBroadcastManager.getInstance(Context context)中对应的函数即可。

其大致流程是一样的:

	//1,获取本地广播的管理器
	LocalBroadcastManager instance = LocalBroadcastManager.getInstance(this);
	//2,动态注册广播
	instance.registerReceiver(sendMyBroadCast, filter);
	//3,发送广播
	instance.sendBroadcast(intent);

#### 有两点需要注意 ####
#### 1,本地广播的方法并不多, 主要有:  registerReceiver(注册广播) , unregisterReceiver(取消注册广播) , sendBroadcast(发送异步广播) , sendBroadcastSync(发送同步广播)四个方法####
#### 2,本地广播只支持代码中注册的广播, 在manifest中声明的广播是接收不到广播的 ! ####

## 5,Sticky广播 (粘性广播, 即可以短时间滞留)(API 21过时了)##

sticky广播通过Context.sendStickyBroadcast()函数来发送，用此函数发送的广播会一直滞留，当有匹配此广播的广播接收器被注册后，该广播接收器就会收到此条信息。(已经过时了)

使用此函数需要发送广播时，需要获得BROADCAST_STICKY权限

	<uses-permission android:name="android.permission.BROADCAST_STICKY"/>

sendStickyBroadcast只保留最后一条广播，并且一直保留下去，这样即使已经有广播接收器处理了该广播，当再有匹配的广播接收器被注册时，此广播仍会被接收。如果你只想处理一遍该广播，可以通过removeStickyBroadcast()函数来实现。

>ps:目前由于一些安全问题，系统已经不建议使用 Sticky broadcasts。

## 6,系统广播 ##

系统中也会有很多自带的广播，当符合一定条件时，系统会发送一些定义好的广播，比如：重启、充电、来电电话等等。我们可以通过action属性来监听我们的系统广播

常用的广播action属性有

	屏幕被关闭之后的广播：Intent.ACTION_SCREEN_OFF(只能通过代码注册)
	屏幕被打开之后的广播：Intent.ACTION_SCREEN_ON(只能通过代码注册)
	充电状态，或者电池的电量发生变化：Intent.ACTION_BATTERY_CHANGED(只能通过代码注册)
	关闭或打开飞行模式时的广播：Intent.ACTION_AIRPLANE_MODE_CHANGED(可静态)
	表示电池电量低：Intent.ACTION_BATTERY_LOW(可以静态注册)
	表示电池电量充足，即电池电量饱满时会发出广播：Intent.ACTION_BATTERY_OKAY(可静态)
	按下照相时的拍照按键(硬件按键)时发出的广播：Intent.ACTION_CAMERA_BUTTON(可静态)
	时间变化的广播: Intent.ACTION_TIME_CHANGED(只能通过代码注册)
	时间变化广播:  ACTION_TIME_CHANGED(时间被设置),ACTION_DATE_CHANGED(日期变化)(这两个可静态)
				ACTION_TIME_TICK(分钟变化,这个需要通过代码动态注册)
	开机广播: ACTION_BOOT_COMPLETED
	安装app:ACTION_PACKAGE_INSTALL ,安装app完成;ACTION_PACKAGE_ADDED
	位置变化: ACTION_LOCALE_CHANGED



