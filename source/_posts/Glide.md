---

title: Glide学习
date: 2018-05-16 13:54:08
tags: Android,Glide
categories: 
    - Android
    - 图片加载
    - 学习总结

---

### 第一部分是 glide 的使用以及和 picasso 的对比
### 第二部分是 glide 与 fresco 的对比

### 1,glide库的地址
[bumptech/glide](https://github.com/bumptech/glide)

<!-- more -->

[Android 图片加载框架最全解析（一），Glide的基本用法](http://blog.csdn.net/guolin_blog/article/details/53759439)

Glide是在Picasso基础上进行了优化改进, 所以这两个是没得挑的..Fresco是fackbook出品的,也很不错,比较

[网络加载框架那个好?](http://stackoverflow.com/questions/29363321/picasso-v-s-imageloader-v-s-fresco-vs-glide)
里面说是 fresco相比较其他的图片加载框架有以下优势:

1,理解可能不准确,说是存储位置不在内存中或者内存占用极小,可以极大的减少了outofmemoryerror的风险,同时gc调用次数减少可以提高app性能

2,图片加载可以表示成进度条, 可以像浏览器那样,体验好

3,图片可以在周边任意裁剪,而不仅仅是居中显示 ,这个非常好用

4,图片可以本地调整大小,可以减少下载时候的oom问题

但是其缺点也有: 库比较大, 上手比较难

glide的优点是可以加载 gif图

###2,基础用法:  Glide是 Picasso 的进化版 . 1,内存优化更好 .2,加载速度更快

Picasso

	Picasso.with(context)
         .load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
         .into(ivImg);

Glide

	Glide.with(context)
         .load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
        .into(ivImg);

Glide更易用，因为Glide的with方法不光接受Context，还接受Activity 和 Fragment，Context会自动的从他们获取。

将Activity/Fragment作为with()参数的好处是：

	图片加载会和Activity/Fragment的生命周期保持一致，比如 Paused状态在暂停加载，
	在Resumed的时候又自动重新加载。所以我建议传参的时候传递Activity 和 Fragment给Glide，而不是Context。

Glide默认的Bitmap格式是RGB_565,比ARGB_8888格式的内存开销要小一半

如果你对默认的RGB_565效果还比较满意，可以不做任何事，但是如果你觉得难以接受，可以创建一个新的GlideModule将Bitmap格式转换到ARGB_8888：

	1. public class GlideConfiguration implements GlideModule {
	3.     @Override
	4.     public void applyOptions(Context context, GlideBuilder builder) {
	5.         // Apply options to the builder here.
	6.         builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
	7.     }
	8.  
	9.     @Override
	10.     public void registerComponents(Context context, Glide glide) {
	11.         // register ModelLoaders here.
	12.     }
	13. }
	
同时在AndroidManifest.xml中将GlideModule定义为meta-data

	<meta-data android:name="com.inthecheesefactory.lab.glidepicasso.GlideConfiguration"
		android:value="GlideModule"/>

原因在于Picasso是加载了全尺寸的图片到内存(这个全尺寸的意思是把原生的图片加载到了内存, 然后根据imageview来进行裁剪)，然后让GPU来实时重绘大小。而Glide加载的大小和ImageView的大小是一致的，因此更小。

一: 指定加载图片的大小
	
	Picasso.with(this)
     .load("http://nuuneoi.com/uploads/source/playstore/cover.jpg")
     .resize(768, 432)
     .into(ivImgPicasso);

	但是问题在于你需要主动计算ImageView的大小，或者说你的ImageView大小是具体的值（而不是wrap_content），你也可以这样：

	Picasso.with(this)
	    .load("http://nuuneoi.com/uploads/source/playstore/cover.jpg")
	    .fit()
	    .centerCrop()
	    .into(ivImgPicasso);

总结:Glide可以自动计算出任意情况下的ImageView大小。

二: 磁盘缓存

Picasso和Glide在磁盘缓存策略上有很大的不同。Picasso缓存的是全尺寸的，而Glide缓存的是跟ImageView尺寸相同的。

Picasso只缓存一个全尺寸的
Glide则不同，它会为每种大小的ImageView缓存 一次。尽管一张图片已经缓存了一次,但是假如你要在另外一个地方再次以不同尺寸显示，需要重新下载，调整成新尺寸的大小，然后将这个尺寸的也缓存起来

例如:假如在第一个页面有一个200x200的ImageView，在第二个页面有一个100x100的ImageView，这两个ImageView本来是要显示同一张图片，却需要下载两次。
不过，你可以改变这种行为，让Glide既缓存全尺寸又缓存其他尺寸：

	Glide.with(this)
      .load("http://nuuneoi.com/uploads/source/playstore/cover.jpg")
      .diskCacheStrategy(DiskCacheStrategy.ALL)
      .into(ivImgGlide);

下次在任何ImageView中加载图片的时候，全尺寸的图片将从缓存中取出，重新调整大小，然后缓存。

Glide的这种方式优点是加载显示非常快。而Picasso的方式则因为需要在显示之前重新调整大小而导致一些延迟，即便你添加了这段代码来让其立即显示

	//Picasso
	 .noFade();

总结: Glide远比Picasso快，虽然需要更大的空间来缓存。(需要更大的空间的原因是picasso只缓存一个最大的图,取的时候把这个图剪裁后进行设置,相对来说速度就比较慢了,而 glide 默认是只缓存imageview大小的图片的,但是获取其他大小的图片的时候却需要重新获取了,当然可以设置一个最大的图,所以这个时候已经缓存变大了,并且在加载其他尺寸的图片的时候会将大图剪裁,再把剪裁后的图重新缓存,这样就相当于用了多少不同尺寸的该图就缓存了几次,还多一次大图, 所以缓存会变大但是加载速度很快... 由于picasso至缓存一个所以每次加载都需要剪裁所以速度没有glide快 ,但是现在的手机存储是非常大的,所以 综上所诉 glide更好)

三: 特性    ---  glide 和 Picasso的特性基本一样,但是glide是picasso的升级版

1,确定剪裁的大小  Image Resizing

	1. // Picasso
	2. .resize(300, 200);
	3.  
	4. // Glide
	5. .override(300, 200);

2,一般情况下剪裁方式	Center Cropping

	1. // Picasso
	2. .centerCrop();
	3.  
	4. // Glide
	5. .centerCrop();

3,自定义显示方式,(例如 设置centercrop或者fitcenter)  Transforming

	1. // Picasso
	2. .transform(new CircleTransform())
	3. .bitmapTransform()  ,transform有两个重载方法, 其中的一个会调用bitmaptransform方法, 效果和直接调用bitmaptransform是一样的, 另一个方法会调用new MultiTransformation<ResourceType>(transformations) 用来创建gif图的显示 , 而bitmaptransform就是显示图片而非gif图片
	3.  
	4. // Glide
	5. .transform(new CircleTransform(context))

4,设置占位图或者加载错误图：

	1. // Picasso
	2. .placeholder(R.drawable.placeholder)
	3. .error(R.drawable.imagenotfound)
	4.  
	5. // Glide
	6. .placeholder(R.drawable.placeholder)
	7. .error(R.drawable.imagenotfound)

5,Glide可以加载gif图, 但是picasso 不行

同时因为Glide和Activity/Fragment的生命周期是一致的，因此gif的动画也会自动的随着Activity/Fragment的状态暂停、重放。Glide 的缓存在gif这里也是一样，调整大小然后缓存。

但是Glide 动画会消费太多的内存，因此谨慎使用。

除了gif动画之外，Glide还可以将任何的本地视频解码成一张静态图片。

还有一个特性是可以配置图片显示的动画，而Picasso只有一种动画：fading in

最后一个是可以使用thumbnail()产生一个加载图片的thumbnail。

其实还有一些特性，不过不是非常重要，比如将图像转换成字节数组等。(这样就可以进行load百分比的显示了)

##配置

有许多可以配置的选项，比如大小，缓存的磁盘位置，最大缓存空间，位图格式等等。可以在这个页面查看这些配置Configuration 。

#总结

Glide和Picasso都是非常完美的库。Glide加载图像以及磁盘缓存的方式都要优于Picasso，速度更快，并且Glide更有利于减少OutOfMemoryError的发生，GIF动画是Glide的杀手锏。不过Picasso的图片质量更高。
你更喜欢哪个呢？
虽然我使用了很长时间的Picasso，但是我得承认现在我更喜欢Glide。我的建议是使用Glide，但是将Bitmap格式换成 ARGB_8888、让Glide缓存同时缓存全尺寸和改变尺寸两种。

#Fresco vs Glide

地址见: [网络加载图片对比（Fresco/Glide）](http://www.jianshu.com/p/6729dc17586b)

最后面有[Fresco中文官网](http://fresco-cn.org/docs/caching.html)

和[Glide 教程](https://futurestud.io/blog/glide-getting-started)

### fresco的功能 ###
![](http://upload-images.jianshu.io/upload_images/1802256-8baeb4e124e62d8f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面介绍一些项目中经常用到的功能，与glide中的对比
#### 圆角， 圆形 

fresco实现

	public void setRoundImageSrc(SimpleDraweeView draweeView, String src, float radius){
	    RoundingParams roundingParams = RoundingParams.fromCornersRadius(radius);
	    draweeView.setHierarchy(
	            new GenericDraweeHierarchyBuilder(draweeView.getResources())
	            .setRoundingParams(roundingParams)
	            .build());
	    draweeView.setImageURI(Uri.parse(src));
	｝

glide实现

	需要自己实现圆角，继承自BitmapTransformation操作bitmap对象实现(圆形同理)：

	public static class GlideRoundTransform extends BitmapTransformation {

	    private static float radius = 0f;
	
	    public GlideRoundTransform(Context context) {
	        this(context, 4);
	    }
	
	    public GlideRoundTransform(Context context, int dp) {
	        super(context);
	        this.radius = Resources.getSystem().getDisplayMetrics().density * dp;
	    }
	
	    @Override 
	    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
	        return roundCrop(pool, toTransform);
	    }
	
	    private static Bitmap roundCrop(BitmapPool pool, Bitmap source) {
	        if (source == null) return null;
	
	        Bitmap result = pool.get(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
	        if (result == null) {
	            result = Bitmap.createBitmap(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
	        }
	
	        Canvas canvas = new Canvas(result);
	        Paint paint = new Paint();
	        paint.setShader(new BitmapShader(source, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
	        paint.setAntiAlias(true);
	        RectF rectF = new RectF(0f, 0f, source.getWidth(), source.getHeight());
	        canvas.drawRoundRect(rectF, radius, radius, paint);
	        return result;
	    }
	
	    @Override public String getId() {
	        return getClass().getName() + Math.round(radius);
	    }
	}

	//使用
	Glide.with(context).load(imageUrl).transform(new GlideRoundTransform(context)).into(imageView)
	//注意：使用了transform以后，就不能使用centercrop，fitcenter等方法

#### 缓存
Fresco缓存也是一大亮点， 三级缓存，分别是 Bitmap缓存，未解码图片缓存， 文件缓存。

这里提一点Bitmap缓存：在5.0以下系统，Bitmap缓存位于ashmem，这样Bitmap对象的创建和释放将不会引发GC，更少的GC会使你的APP运行得更加流畅。5.0及其以上系统，相比之下，内存管理有了很大改进，所以Bitmap缓存直接位于Java的heap上。

另外，磁盘缓存还可以通过代码来设置不同手机的缓存容量：

	public void initFresco(Context context, String diskCacheUniqueName){
	    DiskCacheConfig diskCacheConfig = DiskCacheConfig.newBuilder(context)
	            .setMaxCacheSize(DISK_CACHE_SIZE_HIGH)
	            .setMaxCacheSizeOnLowDiskSpace(DISK_CACHE_SIZE_LOW)
	            .setMaxCacheSizeOnVeryLowDiskSpace(DISK_CACHE_SIZE_VERY_LOW)
	            .build();
	
	    ImagePipelineConfig config = ImagePipelineConfig.newBuilder(context)
	            .setMainDiskCacheConfig(diskCacheConfig)
	            .build();
	    Fresco.initialize(context, config);
	}

Glide缓存

Glide虽然只有内存和磁盘缓存，在性能上比不上Fresco；

但他也有另外的优点， Fresco缓存的时候，只会缓存原始图像，(这点和Picasso 是一样的,实际用的时候由于剪裁方式就没有glide加载速度快了) .

而Glide则会根据ImageView控件尺寸获得对应的大小的bitmap来展示，从而缓存也可以针对不同的对象：原始图像（source），结果图像(result); 可以通过.diskCacheStrategy()方法设置(这样会增加缓存的空间,但是这个并不重要);

	public enum DiskCacheStrategy {    
	  /** Caches with both {@link #SOURCE} and {@link #RESULT}. */    
	  ALL(true, true),    
	  /** Saves no data to cache. */    
	  NONE(false, false),    
	  /** Saves just the original data to cache. */    
	  SOURCE(true, false),    
	  /** Saves the media item after all transformations to cache. */    
	  RESULT(false, true);
	}

####bitmap操作

Glide与Picasso类似，通过简单的方法即可获得网络图片的bitmap对象：

	Bitmap myBitmap = Glide.with(applicationContext)  
	    .load(yourUrl)  	//地址
	    .asBitmap() //必须,表示作为bitmap来获取
	    .centerCrop()  //加载的方式
	    .into(500, 500)  //加载的大小(结合上面的表示在图的中间截取 500*500的大小)
	    .get()	//通过这个方法来实际获取bitmap对象

相反，Fresco要获取bitmap更加复杂， 而且使用起来也并不是那么顺畅。

首先，Fresco为了更好地管理bitmap 对象（bitmap对象申请和释放会引起频繁的GC操作，从而引起界面卡顿）， 引入了可关闭的引用（CloseableReference）, 持有者在离开作用域的时候需要关闭该引用，而我们要获取的bitmap 对象就是可关闭的引用。

也就是说，我们不能像上面Glide那样把bitmap 对象取出来传递给其它地方使用， 只能在Fresco提供的作用域范围内使用，代码如下：
	
	public void setDataSubscriber(Context context, Uri uri, int width, int height){
	    DataSubscriber dataSubscriber = new BaseDataSubscriber<CloseableReference<CloseableBitmap>>() {
	        @Override
	        public void onNewResultImpl(
	                DataSource<CloseableReference<CloseableBitmap>> dataSource) {
	            if (!dataSource.isFinished()) {
	                return;
	            }
	            CloseableReference<CloseableBitmap> imageReference = dataSource.getResult();
	            if (imageReference != null) {
	                final CloseableReference<CloseableBitmap> closeableReference = imageReference.clone();
	                try {
	                    CloseableBitmap closeableBitmap = closeableReference.get();
	                    Bitmap bitmap  = closeableBitmap.getUnderlyingBitmap();
	                    if(bitmap != null && !bitmap.isRecycled()) {
	                        //you can use bitmap here
	                    }
	                } finally {
	                    imageReference.close();
	                    closeableReference.close();
	                }
	            }
	        }
	        @Override
	        public void onFailureImpl(DataSource dataSource) {
	            Throwable throwable = dataSource.getFailureCause();
	            // handle failure
	        }
	    };
	    getBitmap(context, uri, width, height, dataSubscriber);
	}

在实际使用过程中， 如果只有在作用域范围操作bitmap，明显不能满足需求。
项目中使用的方式是获取缓存的文件对象：

	//同样在DataSubscriber中获取
	FileBinaryResource resource = (FileBinaryResource) Fresco.getImagePipelineFactory().getMainFileCache().getResource(new SimpleCacheKey(url));
	if (resource != null && resource.getFile() != null) {           
	    setImage(ImageSource.uri(Uri.fromFile(resource.getFile())));
	}

##其他

除了以上内容，Fresco还具备以下一些常用的，但Glide没有的功能：

1，SimpleDraweeView控件可以指定图片的宽高比例（app:viewAspectRatio），对于手机适配非常重要;

2，图片加载进度；

3，先加载小尺寸图片，再加载大尺寸的（Glide只有占位图）；

##性能

除了在功能上对比， 网络图片显示是非常耗性能的， 下面就针对图片质量，内存使用等情况来对比。

##Glide的使用
			Glide
                .with(this)
                .load(url)
                .placeholder(R.drawable.vip_center_item_1)
				.diskCacheStrategy(DiskCacheStrategy.ALL)
                .bitmapTransform(new GlideRoundTransform(this))
                .centerCrop()
                .into(result);

配合 [Glide的 图形裁剪库可以完美的设置图片](https://github.com/wasabeef/glide-transformations)

			Glide
                .with(context)
                .load(path)
                .placeholder(R.drawable.vip_center_item_1)
                .diskCacheStrategy(DiskCacheStrategy.ALL)
                .bitmapTransform(new RoundedCornersTransformation(context, 5, 1))
                .centerCrop()
                .into(imageView);




	


