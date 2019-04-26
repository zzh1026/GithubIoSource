---
title: Android 设计模式：（一）面向对象的六大原则
comments: false
date: 2018-06-11 17:00:07
tags:
categories:
    - Android
    - 架构
    - 设计模式（转载）
    
---

# 声明 #

**本文为转载文章！尊重原创的劳动果实，严禁剽窃**

本文转载于：[https://www.jianshu.com/p/c9263bdc1ffe](https://www.jianshu.com/p/c9263bdc1ffe "Android 设计模式：（一）面向对象的六大原则")

出自于：[肖丹晨的博客](https://www.jianshu.com/u/d43d948bef39)

本篇为设计模式系列的第一篇： **面向对象的六大原则**

**本文主要用于个人学习回顾使用**

<!-- more -->

# 优化代码的第一步——单一职责原则 #

**单一职责原则（SRP）:** 就一个类而言，应该仅有一个引起他变化的原因。简单来说，一个类应该是一组**高度相关**的函数，数据的封装。

很抽象的概念是不是？别管他，举个栗子：

**开发一个图片加载器（ImageLoader），要求能够实现图片的下载加载，并能将图片缓存起来。**

## 屌丝程序猿小明 ##

**源码**

```
/**
 * 图片加载类
 */
public class ImageLoader{
  //图片缓存
  LruCache<Strig, Bitmap> mImageCache;
  
  public ImageLoader(){
    initImageCatche();
  }
  
  //初始化缓存：mImageCache
  private void initImageCache(){...}
  
  //加载图片
  public void displayImage(String url, ImageView imageView){...}
  
  //下载图片
  public Bitmap downloadImage(String ImgUrl){...}
}
```

**解析**
小明的ImageLoader耦合太严重，简直没有设计可言，更不要说扩展性，灵活性。所有的功能都写在一个类里，随着功能的增多，ImageLoader会越来越重，越来越臃肿复杂。整个图片加载系统就越来越脆弱。

## 装逼程序猿小民 ##

**源码**

```
/**
 * 图片缓存类
 */
public class ImageCache{
  //图片缓存
  LruCache<Strig, Bitmap> mImageCache;
  
  public ImageCache(){
    initImageCatche();
  }
  
  //初始化缓存：mImageCache
  private void initImageCache(){...}
  
  //缓存图片
  public void put(String url, Bitmap bitmap){...}
  
  //获取图片
  public Bitmap get(String url){...}
}
```
```
/**
 * 图片加载类
 */
public class ImageLoader{
  //图片缓存
  LruCache<Strig, Bitmap> mImageCache = new ImageCache();
  
  //加载图片
  public void displayImage(String url, ImageView imageView){...}
  
  //下载图片
  public Bitmap downloadImage(String ImgUrl){...}
}
```

**解析**

小民将小明的版本一份为二：ImageLoader，ImageCache。ImageLoader只负责图片的加载和下载逻辑；ImageCache只负责图片的缓存逻辑。这样架构更加清晰，功能模块的耦合性更加低，相互间的影响更小。

## 总结 ##

从上面的例子中可以初步体会到，单一职责所表达的用意就是“单一”二字。我们设计类的时候，一定要仔细考虑如何划分类的职责和函数的功能。

正如前文所说，一个类应当是一组高度相关的函数和数据的组合，即一个类应当高内聚，低耦合。


# 让程序更稳定、更灵活——开闭原则 #

**开闭原则（OCP）：** 软件中的对象（类，模块，数据等）对扩展是开放的，对于修改是封闭的。换句话就是，程序一旦开发完成，程序中一个类的实现只应该因错误而被修改，新的或者改变的特性应该通过新建不同的类来实现，新建的类可以通过继承的方式来重用原来的代码。

## 屌丝程序猿小明 ##

小明设计DiskCache类，将图片缓存到SD卡中。

**源码**

```
public class DiskCache{
  static String cacheDir = "sdcard/cache/";
  
  //从本地缓存中获取图片
  public Bitmap get(String localUri){...}
  
  //将图片缓存到本地
  public void put(String localUri, Bitmap bmp){...}
}
```
因为需要将图片缓存到SD卡中，所以小明需要修改ImageLoader.java代码：

```
public class ImageLoader{
  ...
  //SD卡缓存
  DiskCache mDiskCache = new DiskCache();
  //是否适用SD卡缓存
  boolean isUseDiskCache = false;
  ...
  //修改加载图片中的缓存策略
  public void displayImage(String url, ImageView imageView){
    //判读使用哪种缓存
    Bitmap bmp = isUseDiskCache?mDiskCache.get(url):mImageCache.get(url);
    ...
  }
  
  //Added:设置是否使用本地缓存加载
  public void useDiskCache(boolean useDiskCache){...}
}
```
通过useDiskCache()方法可以方便的让用户设置缓存方式，小明很开心啊。后来发现这种设计明显有问题，那就是用户只能使用内存缓存和本地缓存的一种，所以小明新建一个双缓存类，实现先内存再本地的加载策略。

```
public class DoubleCache{
  ImageCache mMenoryCache = new ImageCache();
  DiskCache mDiskCache = new DiskCache();
  
  //先从内存中获取图片，获取不到再从SD获取图片
  public Bitmap get(String url){...}
  
  //将图片缓存到内存和SD中
  public void put(String url,Bitmap bmp){...}
}
```

小明需要对ImageLoader更新：

```
public class ImagerLoader{
  ...
  //双缓存
  DoubleCache mDoubleCache = new DoubleCache();
  //使用双缓存
  boolean isUseDoubleCache = false;
  ...
  
  //Modified:修改加载图片策略
  public void dispalyImage()(String url, ImageView imageView){
    Bitmap bmp = null;
    if(isUseDoubleCache){
      bmp = mDoubleCache.get(url);
    }else if(isUseDiskCache){
      bmp = mDiskCache.get(url);
    }else{
     bmp = mImageCache.get(url); 
    }
    ...
  }
  
  //Added:设置是否使用双缓存加载
  public void useDoubleCache(boolean useDoubleCache){...}
}
```

**解析**

小明每次加入新的缓存策略都需要修改ImageLoader类，然后通过if-else语句进行逻辑控制。随着类似这些逻辑的加入，ImageLoader的代码变得越来越臃肿，脆弱。如果一不小心写错了某个if-else的条件，则需要花费大量的时间来排除错误。
最重要的是，ImageLoader的可扩展性差，用户无法自己注入自定义实现的缓存策略。可扩展性是一个框架最重要的特性。
小明的方案很让人郁闷：一扩展，就要修改ImageLoader，一修改就容易出bug。

## 装逼程序猿小民 ##

软件的对象对于扩展应该是开放的，但是对于修改应该是封闭的。也就是说，软件的变化更新应该尽量通过扩展的方式实现，而不是通过修改自己原来的代码实现。
喜爱装逼的小民熟知关闭原则的真谛，他重构了小明的框架：
![](https://upload-images.jianshu.io/upload_images/8104568-2a989334f31aad0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/643)

**源码**

首先定义了缓存策略接口

```
/**
 * 缓存策略接口类
 */
public interface ImageCache{
  public Bitmap get(String url);
  public void put(String url, Bitmap bmp);
}
```

实现缓存策略的子类（缓存策略扩展）

```
//内存缓存策略
public class MemoryCache implements ImageCache{...}
 
//SD缓存策略
public class DiskCache implements ImageCache{...}
 
//双缓存策略
public class DoubleCache implements ImageCache{...}
```
ImageLoader: 图片加载器

```
public class ImageLoader{
  //图片缓存策略
  ImageCache mImageCache = new MemoryCache();
  
  //注入缓存实现
  public setImageCache(ImageCache imageCache){
    mImageCache = imageCache;
  }
  
  //加载图片
  public void displayImage(String url, ImageView imageView){
    Bitmap bmp = mImageCache.get(url);
    if(null != bmp){
      imageView.setImageBitmap(bmp);
      return;
    }
    
    //图片没有缓存，提交到线程池中下载图片
    submitLoadRequest(url,imageView);
  }
  
  private void submitLoadRequest(String url, ImageView imageView){
    ...
    Bitmap bmp = downloadImage(url);
    ...
    imageView.setImageBitmap(bmp);
    ...
    mImageCache.put(url,bmp);
    ...
  }
  
  //下载图片
  public Bitmap downloadImage(String ImgUrl){...}
}
```

## 总结 ##

开闭原则（OCP）指导我们，当软件需求发生变化时，应该**尽量通过扩展**的方式来实现变化，而不是通过修改已有的代码来实现。在软件设计初期，最好能够好好的考量一下可扩展能力的设计。

# 构建扩展性更好的系统——里氏替换原则 #

**里氏替换原则（LSP）:** 所有引用基类的地方，都必须能够透明的使用其子类的对象。说的直白一点就是**多态和抽象**。

里氏替换原则的核心原理是抽象，抽象又依赖于继承这个特性。那么什么是抽象？按照我的理解说的直白点，就是基类，说的再具象点就是虚基类或者接口。

所以，此小节的标题我们完全可以翻译为：利用接口编程思想，构建出具有良好扩展性的系统。这里的接口不单单是指interface，也包括abstract class 和base class。

# 让项目拥有变化的能力——依赖倒置原则 #

**依赖倒置原则（DIP）：**

- 高层模块不应依赖底层模块，两者都应该依赖其抽象。
- 抽象不应该依赖细节。
- 细节应该依赖抽象。

在java语言中，抽象就是指接口或抽象类，两者都是不能直接被实例化的。细节就是实现类，实现接口或继承抽象类而产生的类就是细节，其特点就是，可以直接被实例化。高层模块就是调用端，低层模块就是具体实现类。

DIP在java语言中的表现就是：**模块间的依赖通过抽象发生，实现类之间不发生直接的依赖关系，其依赖关系是通过接口或者抽象类产生的。**简而言之就是**面向接口编程**或者**面向抽象编程**。

接着那上面的代码举例：

## 屌丝程序猿小明 ##

**源码**

```
/**
 * 图片加载类
 */
public class ImageLoader{
  //内存缓存（直接依赖细节）
  MemoryCache mMemoryCache = new MemoryCache();
  ...
}
```

**解析**

高层模块ImageLoader直接依赖低层模块MemoryCache，直接将二者耦合。一方面不容易扩展，另一方面，低层模块MemoryCache修改时很有可能还要修改高层模块ImageLoader，这就违反了开闭原则。

## 装逼程序猿小民 ##

**源码**

```
public class ImageLoader{
  //图片缓存策略(依赖抽象--接口)
  ImageCache mImageCache = new MemoryCache();
  
  //注入缓存实现（依赖注入）
  public setImageCache(ImageCache imageCache){
    mImageCache = imageCache;
  }
  ...
}
```

**解析**

高层模块ImageLoader不直接依赖低层模块MemoryCache，而是二者都依赖其抽象接口ImageCache。一方面容易扩展，另一方面，低层模块MemoryCache修改时不会导致修改高层模块ImageLoader。

## 总结 ##

DIP的核心思想就是面向接口或面向抽象编程，为什么要这样做，还需要大家好好的体会。

# 让系统具有更高的灵活性——接口隔离原则 #

**接口隔离原则（ISP）：** 客户端不应该依赖他不需要的接口，类间的依赖关系应该建立在最小的接口上。

接口隔离原则将非常庞大臃肿的接口拆分成更小的和更具体的接口，这样客户将会只需要知道他们感兴趣的接口的方法。ISP的目的是系统解开耦合，从而容易重构、更改和重新部署。

Bob大叔（Robert C Martin）曾将单一职责原则（SRP）、开闭原则（OCP）、里氏替换原则（LSP）、接口隔离原则（ISP）和依赖倒置原则（DIP）这5个原则称为SOLID原则，作为面向对象开发的基本原则。

# 更好的扩展性——迪米特原则 #

**迪米特原则（LOD）：** 也成为最少知识原则。一个对象应对其他对象有最少的了解。通俗的讲，一个类应该对自己需要耦合或者调用的类知道的最少。

还有一个解释就是：只与直接朋友通信。什么是直接朋友呢？两个对象之间的耦合就是朋友关系，如组合、聚合、依赖。

这个概念，太抽象了。
举个例子嘛：通过中介找房子。我们设定的情况为：租客只要求房间的面积和租金，其他一概不管；中介将符合要求的房子都提供给我。

## 屌丝程序猿小明 ##

**源码**

```
/**
 * 房间
 */
public class Room{
  public float area;//面积
  public float price;//租金
  
  public Room(float area,float price){
    this.area = area;
    this.price = price;
  }
  ...
}
 
/**
 * 中介
 */
public class Mediator{
  //中介和房间类耦合
  List<Room> mRooms = new ArrayList<Room>();
  
  public Mediator(){
    //初始化mRooms
  }
  
  public List<Room> getAllRooms(){
    return mRooms;
  }
}
 
/**
 * 租客
 */
public class Tenant{
  public float roomArea;//租客需要的房子面积
  public float roomPrice;//租客承受的租金
  
  //租客和中介耦合
  public void rentRoom(Mediator mediator){
    //租客和房间耦合
    List<Room> rooms = mediator.getAllRooms();
    for(Room room:rooms){
      if(isSuitable(room)){
        System.out.println("租到房子啦"+room);
        break;
      }
    }
  }
  
  private boolean isSuitable(Room room){
    return room.area >= roomArea 
        && room.price <= roomPrice;
  }
}
```

**解析**

从上面的代码中可以看到，Tenant不仅依赖了Mediator类，还需要频繁的与Room类打交道。把处理的逻辑放在Tenant类中，一方面弱化了中介类的功能，另一方面导致了租户与房间类之间的高度耦合。这种三间关系纠扯不清，一旦Room变化，Tenant也需要跟着变化。

这个时候我们就需要分清谁是租客真正的朋友，让租客只和真正的朋友打交道。这里显然是中介类。

依照迪米特原则，Tenant应该之和真正的朋友Mediator打交道。必须要将Room相关的操作从Tenant中移除。看看小民是如何优雅的装逼的：

## 装逼程序猿小民 ##

**源码**

```
/**
 * 房间
 */
public class Room{
  public float area;//面积
  public float price;//租金
  
  public Room(float area,float price){
    this.area = area;
    this.price = price;
  }
  ...
}
 
/**
 * 中介
 */
public class Mediator{
  //中介和房间类耦合
  List<Room> mRooms = new ArrayList<Room>();
  
  public Mediator(){
    //初始化mRooms
  }
  
  public Room rentOut(float area,float price){
    Room vRoom = null;
    for(Room room:mRooms){
      if(isSuitable(area,price,room)){
        vRoom = room;
        break;
      }
    }
    return vRoom;
  }
  
  private boolean isSuitable(float area,float price,Room room){
    return room.area >= area 
        && room.price <= price;
  }
}
 
/**
 * 租客
 */
public class Tenant{
  public float roomArea;//租客需要的房子面积
  public float roomPrice;//租客承受的租金
  
  //租客和中介耦合
  public void rentRoom(Mediator mediator){
    //租客和房间耦合
    Room room = mediator.rentOut(roomArea,roomPrice);
    if(null != room){
      System.out.println("找到合适的房子啦"+room);
    }else{
      System.out.println("没有找到合适的房子");
    }
  }
}
```

**解析**

对比小明和小民的代码，可以清晰的看出代码的解耦。**只与直接朋友通信**，分清朋友是关键。

## 总结 ##

迪米特原则（最少知识原则）的目的是通过尽量的弱化非直接相关类之间的耦合来实现整体功能的解耦和内聚。

至此，面向对象的六大原则已经讲完了。还希望大家好好的理解体会各个原则的真实内涵。仅仅知道六大原则叫啥，用小民的话说就是逼值到不了满分的（哈哈，玩笑话，装逼还是要的）。深入理解六大原则，对后期学习设计模式，理解设计模式的理念会有很好的帮助。