---

title: Retrofit学习
date: 2018-05-16 13:54:08
tags: Retrofit
categories: 学习知识

---

[查看连接 -- 深入浅出 Retrofit http://chuansong.me/n/365421237869](http://chuansong.me/n/365421237869)

## 1,Hello Retrofit	(这里是需要配置两个东西 : 1,baseurl;2,Converter)
关于配置可以查看[Retrofit2 完全解析 探索与okhttp之间的关系](http://chuansong.me/n/365421237869)

#### 1,添加依赖 ####
>compile 'com.squareup.retrofit2:retrofit:2.1.0'

<!-- more -->

#### 2,定义接口 ####
>	
>		`public interface GitHubService {  
		  @GET("users/{user}/repos")
		  Call<String> listRepos(@Path("user") String user);
		}`
path 代表的是路径 {} 里面代表被替换的网址

#### 3,构造Retrofit ####

> 	Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
	.addConverterFactory(ScalarsConverterFactory.create())	//添加转换器, 这是必须的
	.addCallAdapterFactory(RxJavaCallAdapterFactory.create())	//如果和rxjava配合,需要添加这个转换, 其目的是将 Call<Object> 转换成 Observable<Object>
    .build();

#### 4,通过Retrofit的create方法得到所需的对象 ####

>	GitHubService service = retrofit.create(GitHubService.class);

#### 5,使用得到的对象调用方法,这里像是标记一下指令 ####
>	Call<RestaurantResponse> version = service.getVersion();//设置参数

#### 6,标记指令完毕后在合适的时候调用来获取数据 ####

>同步调用	
>	Response<UpdateVersionResponse> execute = version.execute();
>
>异步调用
>	
        version.enqueue(new Callback<UpdateVersionResponse>() {
            @Override
            public void onResponse(Call<UpdateVersionResponse> call, Response<UpdateVersionResponse> response) {
                Log.i("hehe", "成功" + response.toString());
            }

> 
            @Override
            public void onFailure(Call<UpdateVersionResponse> call, Throwable t) {
                Log.i("hehe", "访问失败");
            }
        });

##2,Url配置

####Retrofit 支持的协议包括 GET/POST/PUT/DELETE/HEAD/PATCH，当然你也可以直接用 HTTP 来自定义请求。这些协议均以注解的形式进行配置

###2.1,GET 的用法
> 	@GET("users/{user}/repos")
> 	Call<Object> listRepos(@Path("user") String user);

  这些注解都有一个参数 value，用来配置其路径,但是不能用于后缀的添加

path 是相对路径，baseUrl 是目录形式：

  	path = "apath"，baseUrl = "http://host:port/a/b/"
  	Url = "http://host:port/a/b/apath"

配置共有四种配置方法

配置的时候采取这种配置方式(baseurl最后以"/"结尾, path中不以"/"开头)

###2.2,参数类型
#### 0,Path ####

path用于在url中动态配置url

#### 1,Query & QueryMap(用于GET方法, 会将值拼接到url的后面, get方法与field是互斥的,get方法不能添加field) ####

	@GET("list")
	Call<要转化成的对象> list(@Query("page") int page);

Query 其实就是 Url 中 ‘?’ 后面的 key-value，比如：http://www.println.net/?cate=android

这里的 cate=android 就是一个 Query，而我们在配置它的时候只需要在接口方法中增加一个参数，即可：

	interface PrintlnServer{    
	   @GET("")    
	   Call<Object> cate(@Query("cate") String cate);
	}

QueryMap用来表示多个(方法为)

Query (Call<RestaurantResponse> getVersion(@QueryMap HashMap<String, String> params);)

2,Field & FieldMap(用于POST方法提交表单请求体中的键值对, 此时仍然能在连接后面以Query添加参数)
#### PS: 使用Field的时候是需要 使用@FormUrlEncoded注解的,表示表示请求正文将使用表单网址编码。字段应该声明为参数并注释@Field。 ####
	   @FormUrlEncoded
	   @POST("/")   
	   Call example(
	       @Field("name") String name,
	       @Field("occupation") String occupation);

	其实也很简单，我们只需要定义上面的接口就可以了，我们用 Field 声明了表单的项，这样提交表单就跟普通的函数调用一样简单直接了

如果表单项不确定个数,可以使用FieldMap

Call<RestaurantResponse> getVersion(@FieldMap HashMap<String, String> params);


3,Part & PartMap(用于POST方法上传文件,get也可以使用该参数上传文件)

表示请求正文是多部分的。零件应声明为参数并注释@Part。

	public interface FileUploadService {  
	    @Multipart
	    @POST("upload")    
		Call upload(@Part("description") RequestBody description,
		                              @Part MultipartBody.Part file);
	}
	
如果你需要上传文件，和我们前面的做法类似，定义一个接口方法，需要注意的是，这个方法不再有 @FormUrlEncoded 这个注解，而换成了 @Multipart，后面只需要在参数中增加 Part 就可以了。也许你会问，这里的 Part 和 Field 究竟有什么区别，其实从功能上讲，无非就是客户端向服务端发起请求携带参数的方式不同，并且前者可以携带的参数类型更加丰富，包括数据流。也正是因为这一点，我们可以通过这种方式来上传文件

![](http://read.html5.qq.com/image?src=forum&q=5&r=0&imgflag=7&imageUrl=http://mmbiz.qpic.cn/mmbiz/tnZGrhTk4ddUIqM8VG30mQk1zeiag5gwNOaicj1WZxODJmbeOTZx8RqwatnOzDEX8zRbVPkXYqesZtak7ia0S4GRw/640?wx_fmt=png)

### 4,Converter 让入参和返回类型丰富 ###

#### 4.1, RequestBodyConverter(自定义请求体) ####

	Retrofit 上传文件，这个上传的过程其实。。还是有那么点儿不够简练，我们只是要提供一个文件用于上传，可我们前后构造了三个对象：
		file -- > requestbody -- > multipartbody.part

Retrofit 允许我们自己定义入参和返回的类型，不过，如果这些类型比较特别，我们还需要准备相应的 Converter，也正是因为 Converter 的存在， Retrofit 在入参和返回类型上表现得非常灵活.

###新的上传文件的接口

	public interface FileUploadService {  
	    @Multipart
	    @POST("upload")    
	    Call upload(@Part("description") RequestBody description,        
			        //注意这里的参数 "aFile" 之前是在创建 MultipartBody.Part 的时候传入的
			        @Part("aFile") File file);
	}
	把入参类型改成了我们熟悉的 File，如果你就这么拿去发请求，服务端收到的结果是一个jsonstring(内部默认的是GsonRequestBodyConverter)

所以就只能自己实现一个 FileRequestBodyConverter

	static class FileRequestBodyConverterFactory extends Converter.Factory {    
   	 	@Override
	    public Converter requestBodyConverter(Type type, Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {      
	       return new FileRequestBodyConverter();
	    }
	  }  
	       
	 static class FileRequestBodyConverter implements Converter<File, RequestBody> {    
	    @Override
	    public RequestBody convert(File file) throws IOException {      
	      return RequestBody.create(MediaType.parse("application/otcet-stream"), file);
	    }
	  }

然后在创建 Retrofit 的时候记得配置上它:

	addConverterFactory(new FileRequestBodyConverterFactory())

这样，我们的文件内容就能上传了

#### 4.2 ResponseBodyConverter 	//这个一般也用不到

前面我们为大家简单示例了如何自定义 RequestBodyConverter，对应的，Retrofit 也支持自定义 ResponseBodyConverter。

再来看下我们定义的接口：

	public interface GitHubService {  
	   @GET("users/{user}/repos")
	  Call<> listRepos(@Path("user") String user);
	}

![](http://read.html5.qq.com/image?src=forum&q=5&r=0&imgflag=7&imageUrl=http://mmbiz.qpic.cn/mmbiz/tnZGrhTk4ddUIqM8VG30mQk1zeiag5gwNonMewRmN43HYdsPbic2fmfKkWcdiazvnRCkDs451PmT7w6SqmkaQcZdA/640?wx_fmt=png)

当然，别忘了在构造 Retrofit 的时候添加这个 Converter，这样我们就能够愉快的让接口返回 Result 对象了。

>注意！！Retrofit 在选择合适的 Converter 时，主要依赖于需要转换的对象类型，在添加 Converter 时，注意 Converter 支持的类型的包含关系以及其顺序。

###Retrofit 原理分析

####1,是谁实际上完成了接口请求的处理？



### 方法总结 ###
#### 最基本的配置 ####
	Retrofit retrofit = new Retrofit.Builder()
	        .baseUrl("http://192.168.31.242:8080/springmvc_users/user/")
	        .addConverterFactory(GsonConverterFactory.create())
	        .build();
	IUserBiz userBiz = retrofit.create(IUserBiz.class);
	Call<List<User>> call = userBiz.getUsers();
	call.enqueue(new Callback<List<User>>()
	        {
	            @Override
	            public void onResponse(Call<List<User>> call, Response<List<User>> response)
	            {
	                Log.e(TAG, "normalGet:" + response.body() + "");
	            }
	
	            @Override
	            public void onFailure(Call<List<User>> call, Throwable t)
	            {
	
	            }
	        });
#### 1,一般的get请求 ####
  	`public interface IUserBiz {
		@GET("users")
		Call<List<User>> getUsers();
	}`
	这是最基本的get请求,没有任何参数及其他.
#### 2,动态修改地址的get请求 -- 使用 Path 注解 ####
	public interface IUserBiz {
	    @GET("{username}")
	    Call<User> getUser(@Path("username") String username);
	}

	这是 使用 Path注解来动态修改url地址的get请求, 但是这个path只能用于修改url而不能用作修改后面的参数, 相当于url中的占位符.
#### 3,查询参数的设置 -- 使用 Query 注解或者 QueryMap 注解 ####
	public interface IUserBiz {
	    @GET("users")
	    Call<List<User>> getUsersBySort(@Query("sortby") String sort);
	}

	eg: http://baseurl/users?sortby=username
		http://baseurl/users?sortby=id

	这样我们就完成了参数的指定，当然相同的方式也适用于POST，只需要把注解修改为@POST即可。
	不同点在于, 这个 Query 注解代表的是在 url后面添加 参数而不是把参数防盗请求体中进行隐藏请求.

#### 4,POST请求体的方式向服务器传入json字符串 -- 使用 Body 注解 ####
	public interface IUserBiz {
	 @POST("add")
	 Call<List<User>> addUser(@Body User user);
	}

	这是通过 Gson 把对象变成Json字符串然后传上去, 不过一般不需要这样,一般都是使用参数传的,所以这种情况的使用情况较少(暂时较少)

#### 5,表单的方式传递键值对 -- 使用 FormUrlEncoded 注解进行标识(Form表示表单形式),然后使用 Field 注解或者 FieldMap注解 ####

	public interface IUserBiz {
	    @POST("login")
	    @FormUrlEncoded
	    Call<User> login(@Field("username") String username, @Field("password") String password);
	}

	这是通过 FormUrlEncoded 进行标识后才能使用 field ,这是使用post的时候在请求体中 添加这些键值对(这种最常用) ,FieldMap注解 则是一个map对象, 表示多个field的参数, (一般用 fieldmap进行post请求,因为 一般请求的时候的参数较多,写多个field不合适.)

#### 6,单文件上传 -- 使用 Multipart 注解进行标识,然后使用 Part 注解或者 PartMap注解 ####
@part可以当成@field来使用,因为 @part是特殊的@field, @part比 @field多支持了文件的类型
	
	文件上传应有的形式: 
		Content-Disposition: form-data; name="file"；filename="test.jpg"
	普通使用Part注解添加的file:
		Content-Disposition: form-data; name="file"

	所以中心思想就是把  file 替换成 file"；filename="test.jpg 通过拼接字符串的方式保存文件

	public interface DemoService {
	    @Multipart()
	    @POST("api/files")
	    Call<ResponseInfo> uploadFile(@Part("file\";filename=\"test.jpg") RequestBody photo);
	}



	public interface IUserBiz {
	    @Multipart
	    @POST("register")
	    Call<User> registerUser(@Part MultipartBody.Part photo, @Part("username") 
			 		RequestBody username, @Part("password") RequestBody password);
	}

	这里@MultiPart的意思就是允许多个@Part了，我们这里使用了3个@Part.
	第一个我们准备上传个文件，使用了MultipartBody.Part类型，
	其余两个均为简单的键值对(这里的键值对说明也可以使用 string,string  的方式 ,上面的 string,requestbody形式并不是必须的 即可以: 
		public interface IUserBiz {
		    @Multipart
		    @POST("register")
		    Call<User> registerUser(@Part MultipartBody.Part photo, @Part("username") 
 					   String username, @Part("password") String password);
		}
	)

	使用的代码为:

	File file = new File(Environment.getExternalStorageDirectory(), "icon.png");
	RequestBody photoRequestBody = RequestBody.create(MediaType.parse("image/png"), file);
	MultipartBody.Part photo = MultipartBody.Part.createFormData("photos", "icon.png", photoRequestBody);
	
	Call<User> call = userBiz.registerUser(photo, RequestBody.create(null, "abc"), RequestBody.create(null, "123"));

#### 7,多文件上传@PartMap -- 使用 PartMap注解 ####

	public interface IUserBiz {
	     @Multipart
	     @POST("register")
	      Call<User> registerUser(
 				@PartMap Map<String, RequestBody> params, 
				@Part("password") RequestBody password);
	}

	这里使用了一个新的注解@PartMap，这个注解用于标识一个Map，Map的key为String类型，代表上传的键值对的key(与服务器接受的key对应),value即为RequestBody，有点类似@Part的封装版本。

执行代码:

	File file = new File(Environment.getExternalStorageDirectory(), "messenger_01.png");
    RequestBody photo = RequestBody.create(MediaType.parse("image/png", file);
	Map<String,RequestBody> photos = new HashMap<>();
	photos.put("photos\"; filename=\"icon.png", photo);
				 "file\"; filename=\"test.jpg"
	photos.put("username",  RequestBody.create(null, "abc"));
	
	Call<User> call = userBiz.registerUser(photos, RequestBody.create(null, "123"));

	可以看到，可以在Map中put进一个或多个文件，键值对等，当然你也可以分开，
	单独的键值对也可以使用 @Part，这里又看到设置文件的时候，相对应的key很奇怪，
	例如上例"photos\"; filename=\"icon.png",前面的photos就是与服务器对应的key，
	后面filename是服务器得到的文件名，ok，参数虽然奇怪，但是也可以动态的设置文件名，不太影响使用

	这个的优势是可以动态的修改名字了,(因为 使用Part注解 value是写死的,所以值没法改,但是使用 PartMap注解 value是创建好传进去的,所以这个名字就可以修改了)

#### 8,下载文件 ####

    public interface IUserBiz {
	    @GET("download")
		Call<ResponseBody> downloadTest();
	}

	然后调用: 

	Call<ResponseBody> call = userBiz.downloadTest();
	call.enqueue(new Callback<ResponseBody>() {
	    @Override
	    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response)
	    {
	        InputStream is = response.body().byteStream();
	        //save file
	    }
	
	    @Override
	    public void onFailure(Call<ResponseBody> call, Throwable t)
	    {
	
	    }
	});



#### field 和 part 和 query 的区别: ####
> field 用于简单键值对的提交, 用在post请求中, 需要进行 @formurlencoude 注解进行标识(get无法使用,这个只能用于post请求体中)
> 
> part 用于post请求中在field的基础上 可以携带文件进行提交(这个只能用于post请求体中)
> 
> query 用于在url后面添加参数, post和get请求均可以使用

[可以查看 直接使用requestbody的上传文件的解决 retrofit#1063](https://github.com/square/retrofit/issues/1063)

@Part("image\"; filename=\"image.jpg\" ") RequestBody image

	public interface ApiInterface {
        @Multipart
        @POST ("/api/Accounts/editaccount")
		Call<User> editUser (
 			@Header("Authorization") String authorization, 
	 		@Part("file\"; filename=\"pp.png") RequestBody file , 
			@Part("FirstName") RequestBody fname, 
			@Part("Id") RequestBody id);
    }

## 配置OkHttpClient ##

1,

	.callFactory(new okhttp3.Call.Factory() {
	                    @Override
	                    public okhttp3.Call newCall(Request request) {
	                        OkHttpClient okHttpClient = new OkHttpClient();
	                        return okHttpClient.newCall(request);
	                    }
	                })

	可以单独写一个OkhttpClient的单例生成类，在这个里面完成你所需的所有的配置，然后将OkhttpClient实例通过方法公布出来，设置给retrofit
	callFactory方法接受一个okhttp3.Call.Factory对象，OkHttpClient即为一个实现类

2,

	.client(okhttpclient)
	
	也可以配置client

	


//转换器 , 请求原始数据转换成对象(addConverterFactory()),一般将该数据转换成json

Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars:2.1.0

Gson: com.squareup.retrofit2:converter-gson:2.1.0

Jackson: com.squareup.retrofit2:converter-jackson:{最新版本号}

Moshi: com.squareup.retrofit2:converter-moshi:{最新版本号}

Protobuf: com.squareup.retrofit2:converter-protobuf:{最新版本号}

Wire: com.squareup.retrofit2:converter-wire:{最新版本号}

Simple XML: com.squareup.retrofit2:converter-simplexml:{最新版本号}

//转换器 , 将返回的Call对象转换成其他 (addCallAdapterFactory()),一般将该数据转换成rxjava


# 注: 添加header的方法 #
该文章见[retrofit 网络请求库 : http://blog.csdn.net/ghost_programmer/article/details/52372065](http://blog.csdn.net/ghost_programmer/article/details/52372065)

## @headers 和 @header ## 
headers是在方法上部声明, 不能动态修改, 不可覆盖 , header 是在方法的参数中代表的, 可以动态设置

## OKhttp  配置的时候在所有的request中添加header   Interceptor ##
官方demo见 [https://github.com/square/okhttp/wiki/Interceptors](https://github.com/square/okhttp/wiki/Interceptors)


	Request request = chain.request();

	Response response = chain.proceed(request);

#### 通过chain的request()方法，可以返回Request对象。通过chain的proceed()方法，可以返回此次请求的响应对象。 ####

#### 对所有的请求都添加请求头 ####

			public okhttp3.Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                // 重写request
                Request requestOverwrite = request.newBuilder().header("User-Agent","Android").build();

                return chain.proceed(requestOverwrite);
            }

#### 同理, 要对所有的请求相应 response 添加header的话 ####

			@Override
            public okhttp3.Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                okhttp3.Response originalResponse = chain.proceed(request);

                return originalResponse.newBuilder().header("Cache-Control","max-age=100").build();
            }





