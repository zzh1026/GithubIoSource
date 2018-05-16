
---

title: Android热修复
date: 2018-05-16 13:54:08
tags: Android,热修复
categories: 学习知识
description: Android热修复学习

---


[热修复——深入浅出原理与实现](https://juejin.im/post/5a0ad2b551882531ba1077a2 "热修复——深入浅出原理与实现")
## 1, 目前较火的热修复方案:

1. 阿里系：DeXposed、andfix：从底层二进制入手（c语言）。
2. 腾讯系：tinker：从java加载机制入手。

## 2, ClassLoader  在jdk 的dalvik.system包下
可以动态加载class文件到内存当中,其作用为:
- 负责将Class加载到JVM中
- 审查每个类由谁加载(父类优先的等级加载机制)
- 将Class字节码重新解析成JVM统一要求的对象格式
- PathClassLoader,只能加载已经安装到Android系统中的apk文件（/data/app目录）,是Android默认使用的类加载器。
- DexClassLoader,可以加载任意目录下的dex/jar/apk/zip文件,一般的补丁都是通过这种方式加载的

#### 1,PathClassLoader与DexClassLoader的代码区别 

- PathClassLoader与DexClassLoader都继承于BaseDexClassLoader。
- PathClassLoader与DexClassLoader在构造函数中都调用了父类的构造函数，但DexClassLoader多传了一个optimizedDirectory。

![Android的类加载器](https://user-gold-cdn.xitu.io/2017/11/14/15fba469740ee5d3?imageslim)

#### 2,BaseDexClassLoader
BaseDexClassLoader的构造函数:

	public BaseDexClassLoader(String dexPath, File optimizedDirectory, String libraryPath, ClassLoader parent){
        super(parent);
        this.pathList = new DexPathList(this, dexPath, libraryPath, optimizedDirectory);
    }

- dexPath：要加载的程序文件（一般是dex文件，也可以是jar/apk/zip文件）所在目录。
- optimizedDirectory：dex文件的输出目录（因为在加载jar/apk/zip等压缩格式的程序文件时会解压出其中的dex文件，该目录就是专门用于存放这些被解压出来的dex文件的）。
- libraryPath：加载程序文件时需要用到的库路径。
- parent：父加载器

> 类加载器加载到的class的方法为 findClass() , 在 BaseDexClassLoader有重写findClass(),
> 该方法主要是通过DexPathList对象（pathList）的findClass()方法来获取class的，而这个DexPathList对象恰好在之前的BaseDexClassLoader构造函数中就已经被创建好了。

![BaseDexClassLoader的内容](https://user-gold-cdn.xitu.io/2017/11/14/15fba4697333434b?imageslim)

#### 3,DexPathList
- DexPathList的构造函数是将一个个的程序文件（可能是dex、apk、jar、zip）封装成一个个Element对象，最后添加到Element集合中。
- DexPathList的findClass()方法很简单，就只是对Element数组进行遍历，一旦找到类名与name相同的类时，就直接返回这个class，找不到则返回null。

![DexPathList内容](https://user-gold-cdn.xitu.io/2017/11/14/15fba46975665b3b?imageslim)

## 3, 热修复的实现原理

经过对PathClassLoader、DexClassLoader、BaseDexClassLoader、DexPathList的分析，我们知道，安卓的类加载器在加载一个类时会先从自身DexPathList对象中的Element数组中获取（Element[] dexElements）到对应的类，之后再加载。采用的是数组遍历的方式，不过注意，遍历出来的是一个个的dex文件。

在for循环中，首先遍历出来的是dex文件，然后再是从dex文件中获取class，所以，我们只要让修复好的class打包成一个dex文件，放于Element数组的第一个元素，这样就能保证获取到的class是最新修复好的class了（当然，有bug的class也是存在的，不过是放在了Element数组的最后一个元素中，所以没有机会被拿到而已）。

![dex修复原理图](https://user-gold-cdn.xitu.io/2017/11/14/15fba469739e5b36?imageslim)

## 4, 热修复的简单实现

#### 1, 重新将修改后的java文件编译成class文件
![生成新的dex文件过程](https://user-gold-cdn.xitu.io/2017/11/14/15fba46977048892?imageslim)

将修复好的class文件复制到其他地方，例如桌面上的dex文件夹中。需要注意的是，在复制这个class文件时，需要把它所在的完整包目录一起复制。

#### 2, 通过build-tools 中的  dx指令程序 将class文件打包成dex文件
dx指令也需要有程序来提供，它就在Android SDK的build-tools目录下各个Android版本目录之中。

>  dx指令
>  dx --dex --output=dex文件完整路径 (空格) 要打包的完整class文件所在目录
>  
>  dx --dex --output=C:\Users\Administrator\Desktop\dex\classes2.dex C:\Users\Administrator\Desktop\dex
>  表示 把 desktop\dex 文件夹下的文件 生成 classes2.dex 文件
>  





