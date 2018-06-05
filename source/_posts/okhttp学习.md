---

title: OKhttp学习
date: 2018-05-16 13:54:08
tags: OKhttp
categories:
     - Android
     - 学习知识

---

1,http请求和响应步骤(7步)
	
	1,经过三次握手简历tcp连接,在HTTP工作开始之前, Web浏览器首先要通过网络与Web服务器建立连接, 
		该连接是通过TCP来完成的, 该协议与IP协议共同构建Internet, 即著名的TCP/IP协议族

	2,web浏览器向web服务器发送请求命令 (请求方法,请求协议版本)
		例如: GET/sample/hello.jsp HTTP/1.1
		
<!-- more -->

	3,web浏览器发送请求头信息 
		浏览器发送其请求命令之后, 还要以头信息的形式向Web服务器发送一些别的信息, 
		这些信息用来描述浏览器自己. 之后浏览器发送了一空白行来通知服务器, 
		表示它已经结束了该头信息的发送. 若是post请求, 还会在发送完请求头信息之后发送请求体

	4,Web服务器应答(对应于请求命令, 这是给予应答)
		客户机向服务器发出请求后, 服务器会向客户机回送应答. 例如: HTTP/1.1 200 OK
		应答的第一部分是协议的版本号和应答状态码

	5, Web服务器发送应答头信息(对应于 浏览器发送请求头信息, 这是应答头信息)
		正如客户端会随同请求发送关于自身的信息一样,服务器也会随同应答向用户发送关于它自己的数据及被请求的文档. 最后以一个空白行来表示头信息发送到此结束

	6,Web服务器向浏览器发送数据
		Web服务器向浏览器发送头信息后, 它就以Content-Type应答头信息所描述的格式发送用户所请求的实际数据

	7,Web服务器关闭TCP连接
		一般情况下, 一旦Web服务器向浏览器发送了请求数据, 它就要关闭TCP连接. 
		如果浏览器或者服务器在其头信息加入了这行代码 : Connection:keep-alive;
		TCP连接在发送后将仍然保持打开状态. 于是, 浏览器可以继续通过相同的连接发送请求. 保持连接节省了为每个请求建立新连接所需的时间, 还节约了网络带宽.

2,历史上Http请求库优缺点 : HttpURLConnection,HttpClient

3,OkHttp类与http请求响应的映射(okhttp的实现步骤, 3步)

	1,创建okclient(这个是需要一个全局的配置)
	OkHttpClient client = new OkHttpClient.Builder()
                .readTimeout(5, TimeUnit.SECONDS)
                .build();
	
	2,创建请求对象
	OkHttp请求 Request request = new Request.Builder().url("https://github.com/").build();
	
	3,执行请求(同步)
	OkHttp响应 Response response = client.newCall(request).execute();
			client.newCall(request).enqueue(new Callback() {}); (异步)

4,HTTP GET
	
	1,同步GET
		`OkHttpClient client = new OkHttpClient();
		Request request = new Request.Builder().url("http://publicobject.com/helloworld.txt").build();	//创建请求

		Response response = client.newCall(request).execute();	//执行请求

		if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

	    Headers responseHeaders = response.headers();
	    for (int i = 0; i < responseHeaders.size(); i++) {
	      System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
	    }		//获取结果

		`

	OkHttpClient实现了Call.Factory接口, 是Call的工厂类, Call负责发送执行请求和读取响应.

	Request代表Http请求, 通过Request.Builder辅助类来构建.

	client.newCall(request)通过传入一个http request, 返回一个Call调用. 然后执行execute()方法, 同步获得

	Response代表Http请求的响应. response.body()是ResponseBody类, 代表响应体, 
		可以通过responseBody.string()获得字符串的表达形式, 或responseBody.bytes()获得字节数组的表达形式, 
		这两种形式都会把文档加入到内存. 也可以通过responseBody.charStream()和responseBody.byteStream()返回流来处理

	2,异步GET
		
		`OkHttpClient client = new OkHttpClient();
		Request request = new Request.Builder().url("http://publicobject.comhelloworld.txt").build();	//创建请求

		client.newCall(request).enqueue(new Callback() {	//异步请求
	      @Override public void onFailure(Request request, Throwable throwable) {
	        throwable.printStackTrace();	//获得结果
	      }
	
	      @Override public void onResponse(Response response) throws IOException {
	        if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);	//获得结果
	
	        Headers responseHeaders = response.headers();
	        for (int i = 0; i < responseHeaders.size(); i++) {
	          System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
	        }
	
	        System.out.println(response.body().string());
	      }
	    });

	onResponse和onFailure这两个方法都是在工作线程中执行的。

5,HTTP POST(该post是在 request请求创建的时候调用post(RequestBody)方法来实现的,该方法通过接收一个requestbody来实现,其他地方(创建client和执行excute等是不受影响的))

**在 ohhttp 的post使用中，常用的有以下一些方式： 包括（1，提交String字符串--不常用）（2，提交一个数据流--不常用）（3，提交一个文件：可以使图片或者文档--偶尔用）（4，提交form标签表单--这个是最常用的，一般或者是get请求或者是post请求都是使用form表单的形式）（5，提交文件和表单的集合--这种比较常用，如修改头像的时候不仅仅是需要一个图片文件，而是需要图片的同时还需要有一个form表单的参数，例如需要对图片作一些说明等）**

json ：application/json

xml：application/xml

png：image/png

jpg： image/jpeg

gif：image/gif

md : text/x-markdown
	
###1,Post方式提交String

	public static final MediaType MEDIA_TYPE_MARKDOWN = MediaType.parse("text/x-markdown; charset=utf-8");		//确定上传的类型

	OkHttpClient client = new OkHttpClient();	//创建一个连接

	String postBody = ""
        + "Releases\n"
        + "--------\n"
        + "\n"
        + " * _1.0_ May 6, 2013\n"
        + " * _1.1_ June 15, 2013\n"
        + " * _1.2_ August 11, 2013\n";		//post必须有请求体

    Request request = new Request.Builder()
        .url("https://api.github.com/markdown/raw")
        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
        .build();		//创建post请求

    Response response = client.newCall(request).execute();	//执行post请求
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());	//如果成功的话请求结果
	`

	Request.Builder的post方法接收一个RequestBody对象。

	RequestBody就是请求体，一般可通过调用该类的5个重载的static的create()方法得到RequestBody对象。
	create()方法第一个参数都是MediaType类型
	create()方法的第二个参数可以是String、File、byte[]或okio.ByteString
	oktttp中create()方法共有5个重载方法，共有三种类型 分别是  File ， ByteString ， Byte[]

	还可以调用RequestBody的writeTo()方法向其写入数据，writeTo()方法一般在用POST发送Stream流的时候使用。

	>注意: 请求体会放置在内存中，所以应该避免用该API发送超过1M的数据。

	2,Post方式提交流

	`
	public static final MediaType MEDIA_TYPE_MARKDOWN
      = MediaType.parse("text/x-markdown; charset=utf-8");		//确定上传的类型

	OkHttpClient client = new OkHttpClient();	//创建一个总开关

	RequestBody requestBody = new RequestBody() {
      @Override public MediaType contentType() {
        return MEDIA_TYPE_MARKDOWN;
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        sink.writeUtf8("Numbers\n");
        sink.writeUtf8("-------\n");
        for (int i = 2; i <= 997; i++) {
          sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
        }
      }

      private String factor(int n) {
        for (int i = 2; i < n; i++) {
          int x = n / i;
          if (x * i == n) return factor(x) + " × " + i;
        }
        return Integer.toString(n);
      }
    };


	Request request = new Request.Builder()
        .url("https://api.github.com/markdown/raw")
        .post(requestBody)
        .build();		//创建post请求

    Response response = client.newCall(request).execute();	//执行post请求
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());	//如果成功的话请求结果
	`

	以流的方式POST提交请求体. 请求体的内容由流写入产生.
	也可以使用BufferedSink.outputStream()来获取. OkHttp的底层对流和字节的操作都是基于Okio库
	`
	
	3,Post方式提交文件.

	`public static final MediaType MEDIA_TYPE_MARKDOWN		//设置MediaType
	  = MediaType.parse("text/x-markdown; charset=utf-8");
	
	private final OkHttpClient client = new OkHttpClient();	//创建连接
	
	public void run() throws Exception {
	    File file = new File("README.md");			//找到文件
	
	    Request request = new Request.Builder()		//创建请求并配置
	        .url("https://api.github.com/markdown/raw")
	        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))
	        .build();
	
	    Response response = client.newCall(request).execute();		//执行
	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	
	    System.out.println(response.body().string());
	}
	`

	4,Post方式提交表单
	
	使用FormEncodingBuilder来构建和HTML<form>标签相同效果的请求体. 键值对将使用一种HTML兼容形式的URL编码来进行编码.

	private final OkHttpClient client = new OkHttpClient();

	  public void run() throws Exception {
	    RequestBody formBody = new FormBody.Builder()
	        .add("search", "Jurassic Park")
	        .build();
	    Request request = new Request.Builder()
	        .url("https://en.wikipedia.org/w/index.php")
	        .post(formBody)
	        .build();
	
	    Response response = client.newCall(request).execute();
	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	
	    System.out.println(response.body().string());
	  }

	5,Post方式提交分块请求

	MultipartBody.Builder可以构建复杂的请求体, 与HTML文件上传形式兼容. 多块请求体中每块请求都是一个请求体, 可以定义自己的请求头. 
	这些请求头可以用来描述这块请求, 例如它的Content-Disposition. 如果Content-Length和Content-Type可用的话, 他们会被自动添加到请求头中.

   [方法是这个地址](http://www.jianshu.com/p/ca8a982a116b)		//这种基本用不到

	我们可以通过Web前端的Form表单上传一个或多个文件，Okhttp也提供了对应的功能，如果我们想同时发送多个Form表单形式的文件，
	就可以使用在post()方法中传入MultipartBody对象。MultipartBody继承自RequestBody，也表示请求体。
	只不过MultipartBody的内部是由多个part组成的，每个part就单独包含了一个RequestBody请求体，
	所以可以把MultipartBody看成是一个RequestBody的数组，而且可以分别给每个RequestBody单独设置请求头。

	> 注意: 这里一般用在上传文件的同时添加参数的时候
	
	private static final String IMGUR_CLIENT_ID = "...";
	  private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");
	
	  private final OkHttpClient client = new OkHttpClient();
	
	  public void run() throws Exception {
	    // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image
	    RequestBody requestBody = new MultipartBody.Builder()
	        .setType(MultipartBody.FORM)
	        .addFormDataPart("title", "Square Logo")
	        .addFormDataPart("image", "logo-square.png",
	            RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
	        .build();
	
	    Request request = new Request.Builder()
	        .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
	        .url("https://api.imgur.com/3/image")
	        .post(requestBody)
	        .build();
	
	    Response response = client.newCall(request).execute();
	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	
	    System.out.println(response.body().string());
	  }

6,其他用法
   [关于相应头的解释见:http://www.jianshu.com/p/2fa728c8b366](http://www.jianshu.com/p/2fa728c8b366)
	
	1,提取响应头

	OkHttp的api
	当写请求头的时候, 使用header(name, value)可以设置唯一的name、value. 如果已经有值, 旧的将被移除, 然后添加新的. 
	使用addHeader(name, value)可以添加多值（添加, 不移除已有的）.

	当读取响应头时, 使用header(name)返回最后出现的name、value. 通常情况这也是唯一的name、value. 如果没有值, 那么header(name)将返回null. 
	如果想读取字段对应的所有值, 使用headers(name)会返回一个list.

	`private final OkHttpClient client = new OkHttpClient();

		public void run() throws Exception {
		    Request request = new Request.Builder()
		        .url("https://api.github.com/repos/square/okhttp/issues")
		        .header("User-Agent", "OkHttp Headers.java")
		        .addHeader("Accept", "application/json; q=0.5")
		        .addHeader("Accept", "application/vnd.github.v3+json")
		        .build();
		
		    Response response = client.newCall(request).execute();
		    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
		
		    System.out.println("Server: " + response.header("Server"));
		    System.out.println("Date: " + response.header("Date"));
		    System.out.println("Vary: " + response.headers("Vary"));
		}`

	2,使用Gson来解析JSON响应

	  private final OkHttpClient client = new OkHttpClient();
	  private final Gson gson = new Gson();
	
	  public void run() throws Exception {
	    Request request = new Request.Builder()
	        .url("https://api.github.com/gists/c2a7c39532239ff261be")
	        .build();
	    Response response = client.newCall(request).execute();
	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	
	    Gist gist = gson.fromJson(response.body().charStream(), Gist.class);
	    for (Map.Entry<String, GistFile> entry : gist.files.entrySet()) {
	      System.out.println(entry.getKey());
	      System.out.println(entry.getValue().content);
	    }
	  }
	
	  static class Gist {
	    Map<String, GistFile> files;
	  }
	
	  static class GistFile {
	    String content;
	  }

	3,响应缓存

	为了缓存响应, 你需要一个你可以读写的缓存目录, 和缓存大小的限制. 这个缓存目录应该是私有的, 不信任的程序应不能读取缓存内容.

	一个缓存目录同时拥有多个缓存访问是错误的. 大多数程序只需要调用一次new OkHttp(), 在第一次调用时配置好缓存, 然后其他地方只需要调用这个实例就可以了. 
	否则两个缓存示例互相干扰, 破坏响应缓存, 而且有可能会导致程序崩溃.

	响应缓存使用HTTP头作为配置. 你可以在请求头中添加Cache-Control: max-stale=3600 , 
	OkHttp缓存会支持. 你的服务通过响应头确定响应缓存多长时间, 例如使用Cache-Control: max-age=9600.

		private final OkHttpClient client;

		public CacheResponse(File cacheDirectory) throws Exception {
		    int cacheSize = 10 * 1024 * 1024; // 10 MiB
		    Cache cache = new Cache(cacheDirectory, cacheSize);
		
		    client = new OkHttpClient();
		    client.setCache(cache);
		}
		
		public void run() throws Exception {
		    Request request = new Request.Builder()
		        .url("http://publicobject.com/helloworld.txt")
		        .build();
		
		    Response response1 = client.newCall(request).execute();
		    if (!response1.isSuccessful()) throw new IOException("Unexpected code " + response1);
		
		    String response1Body = response1.body().string();
		    System.out.println("Response 1 response:          " + response1);
		    System.out.println("Response 1 cache response:    " + response1.cacheResponse());
		    System.out.println("Response 1 network response:  " + response1.networkResponse());
		
		    Response response2 = client.newCall(request).execute();
		    if (!response2.isSuccessful()) throw new IOException("Unexpected code " + response2);
		
		    String response2Body = response2.body().string();
		    System.out.println("Response 2 response:          " + response2);
		    System.out.println("Response 2 cache response:    " + response2.cacheResponse());
		    System.out.println("Response 2 network response:  " + response2.networkResponse());
		
		    System.out.println("Response 2 equals Response 1? " + response1Body.equals(response2Body));
		}

	上面代码，我们对于同一个URL，我们先后发送了两个HTTP请求。
	第一次请求完成后，Okhttp将请求到的结果写入到了缓存目录中，进行了缓存。
	response1.networkResponse()返回了实际的数据，response1.cacheResponse()返回了null，这说明第一次HTTP请求的得到的响应是通过发送实际的网络请求，而不是来自于缓存。
	然后对同一个URL进行了第二次HTTP请求，response2.networkResponse()返回了null，response2.cacheResponse()返回了缓存数据.
	这说明第二次HTTP请求得到的响应来自于缓存，而不是网络请求。

	如果需要阻止response使用缓存, 使用CacheControl.FORCE_NETWORK. 如果需要阻止response使用网络, 使用CacheControl.FORCE_CACHE.

	如果想让某次请求禁用缓存，可以调用 request.cacheControl(CacheControl.FORCE_NETWORK)方法

	如果想强制某次请求使用缓存的结果，可以调用request.cacheControl(CacheControl.FORCE_CACHE)

	警告: 如果你使用FORCE_CACHE, 但是response要求使用网络, OkHttp将会返回一个504 Unsatisfiable Request响应

		1,例如: 

			有些时候, 比如用户刚刚点击刷新按钮, 这时必须跳过缓存, 直接从服务器抓取数据. 为了强制全面刷新, 我们需要添加no-cache指令:
			connection.addRequestProperty("Cache-Control", "no-cache");
			这样就可以强制每次请求直接发送给源服务器, 而不经过本地缓存版本的校验, 常用于需要确认认证的应用和严格要求使用最新数据的应用.
		
		2,
			有时你会想立即显示资源. 这样即使在后台正下载着最新资源, 你的客户端仍然可以先显示原有资源, 毕竟有个东西显示比没有东西显示要好.
			如果需要限制让请求优先使用本地缓存资源, 需要增加only-if-cached指令:
			
			try {
			     connection.addRequestProperty("Cache-Control", "only-if-cached");
			     InputStream cached = connection.getInputStream();
			     // the resource was cached! show it
			  catch (FileNotFoundException e) {
			     // the resource was not cached
			 }
			}

	4,取消一个Call

	使用Call.cancel()可以立即停止掉一个正在执行的call. 
	如果一个线程正在写请求或者读响应, 将会引发IOException. 
	当call没有必要的时候, 使用这个api可以节约网络资源.
	例如当用户离开一个应用时, 不管同步还是异步的call都可以取消

	可以通过tags来同时取消多个请求. 
	当构建请求时, 使用RequestBuilder.tag(tag)来分配一个标签, 之后就可以用OkHttpClient.cancel(tag)来取消所有带有这个tag的call.

		  private final ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

		  private final OkHttpClient client = new OkHttpClient();
		
		  public void run() throws Exception {
		    Request request = new Request.Builder()
		        .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
		        .build();
		
		    final long startNanos = System.nanoTime();
		    final Call call = client.newCall(request);
		
		    // Schedule a job to cancel the call in 1 second.
		    executor.schedule(new Runnable() {
		      @Override public void run() {
		        System.out.printf("%.2f Canceling call.%n", (System.nanoTime() - startNanos) / 1e9f);
		        call.cancel();
		        System.out.printf("%.2f Canceled call.%n", (System.nanoTime() - startNanos) / 1e9f);
		      }
		    }, 1, TimeUnit.SECONDS);
		
		    try {
		      System.out.printf("%.2f Executing call.%n", (System.nanoTime() - startNanos) / 1e9f);
		      Response response = call.execute();
		      System.out.printf("%.2f Call was expected to fail, but completed: %s%n",
		          (System.nanoTime() - startNanos) / 1e9f, response);
		    } catch (IOException e) {
		      System.out.printf("%.2f Call failed as expected: %s%n",
		          (System.nanoTime() - startNanos) / 1e9f, e);
		    }
		  }

	上述请求，服务器端会有两秒的延时，在客户端发出请求1秒之后，请求还未完成，这时候通过cancel方法中止了Call，请求中断，并触发IOException异常。

	5,超时

	没有响应时使用超时结束call. 没有响应的原因可能是客户点链接问题、服务器可用性问题或者这之间的其他东西. OkHttp支持连接超时, 读取超时和写入超时.

	 `private final OkHttpClient client;

	  public ConfigureTimeouts() throws Exception {
	    client = new OkHttpClient.Builder()
	        .connectTimeout(10, TimeUnit.SECONDS)
	        .writeTimeout(10, TimeUnit.SECONDS)
	        .readTimeout(30, TimeUnit.SECONDS)
	        .build();
	  }
	
	  public void run() throws Exception {
	    Request request = new Request.Builder()
	        .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
	        .build();
	
	    Response response = client.newCall(request).execute();
	    System.out.println("Response completed: " + response);
	  }`

	6,每个call的配置

	使用OkHttpClient, 所有的HTTP Client配置包括代理设置、超时设置、缓存设置. 当你需要为单个call改变配置的时候, 调用OkHttpClient.newBuilder(). 
	这个api将会返回一个builder, 这个builder和原始的client共享相同的连接池, 分发器和配置.
	下面的例子中，我们让一个请求是500ms的超时、另一个是3000ms的超时。

		` private final OkHttpClient client = new OkHttpClient();

		  public void run() throws Exception {
		    Request request = new Request.Builder()
		        .url("http://httpbin.org/delay/1") // This URL is served with a 1 second delay.
		        .build();
		
		    try {
		      // Copy to customize OkHttp for this request.
		      OkHttpClient copy = client.newBuilder()
		          .readTimeout(500, TimeUnit.MILLISECONDS)
		          .build();
		
		      Response response = copy.newCall(request).execute();
		      System.out.println("Response 1 succeeded: " + response);
		    } catch (IOException e) {
		      System.out.println("Response 1 failed: " + e);
		    }
		
		    try {
		      // Copy to customize OkHttp for this request.
		      OkHttpClient copy = client.newBuilder()
		          .readTimeout(3000, TimeUnit.MILLISECONDS)
		          .build();
		
		      Response response = copy.newCall(request).execute();
		      System.out.println("Response 2 succeeded: " + response);
		    } catch (IOException e) {
		      System.out.println("Response 2 failed: " + e);
		    }
		  }`

	7,处理验证
		
		1,
[查看连接 -- OkHttp使用完全教程](http://www.jianshu.com/p/ca8a982a116b)

		2,OkHttp认证
[查看连接 -- Android 网络(三) HttpURLConnection OkHttp](http://www.jianshu.com/p/2fa728c8b366)

	8,ResponseBody详细说明:

	通过Response的body()方法可以得到响应体ResponseBody，响应体必须最终要被关闭，否则会导致资源泄露、App运行变慢甚至崩溃。

	ResponseBody和Response都实现了Closeable和AutoCloseable接口，它们都有close()方法，
	Response的close()方法内部直接调用了ResponseBody的close()方法，无论是同步调用execute()还是异步回调onResponse()，
	最终都需要关闭响应体，可以通过如下方法关闭响应体：

	Response.close()
	Response.body().close()
	Response.body().source().close()
	Response.body().charStream().close()
	Response.body().byteString().close()
	Response.body().bytes()
	Response.body().string()

	对于同步调用:  
	
	 Call call = client.newCall(request);
	 try (Response response = call.execute()) {
	   ... // Use the response.
	 }

	也可以在异步回调方法onResponse()中，执行类似的try代码块，
	try()代码块括号中的ResponseBody也实现了Closeable和AutoCloseable接口，
	这样编译器也会在隐式的finally代码块中自动关闭响应体，代码如下所示

	   Call call = client.newCall(request);
	   call.enqueue(new Callback() {
	     public void onResponse(Call call, Response response) throws IOException {
	       try (ResponseBody responseBody = response.body()) {
	         ... // Use the response.
	       }
	     }
	
	     public void onFailure(Call call, IOException e) {
	       ... // Handle the failure.
	     }
	   });
	
	响应体中的数据有可能很大，应该只读取一次响应体的数据。
	调用ResponseBody的bytes()或string()方法会将整个响应体数据写入到内存中，可以通过source()、byteStream()或charStream()进行流式处理。