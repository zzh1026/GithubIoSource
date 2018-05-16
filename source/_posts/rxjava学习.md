---

title: Rxjava学习
date: 2018-05-15 19:36:35
tags: Retrofit
categories: 学习知识
description: Rxjava学习笔记

---

1,创建观察者:  observer  subscriber
	
	Observer 即观察者，它决定事件触发的时候将有怎样的行为。 RxJava 中的 Observer 接口的实现方式：

	Observer<String> observer = new Observer<String>() {
	    @Override
	    public void onNext(String s) {
	        Log.d(tag, "Item: " + s);
	    }
	
	    @Override
	    public void onCompleted() {
	        Log.d(tag, "Completed!");
	    }
	
	    @Override
	    public void onError(Throwable e) {
	        Log.d(tag, "Error!");
	    }
	};

2,创建被观察者: observable

	Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
	    @Override
	    public void call(Subscriber<? super String> subscriber) {
	        subscriber.onNext("Hello");
	        subscriber.onNext("Hi");
	        subscriber.onNext("Aloha");
	        subscriber.onCompleted();
	    }
	});

	可以看到，这里传入了一个 OnSubscribe 对象作为参数。
	OnSubscribe 会被存储在返回的 Observable 对象中，它的作用相当于一个计划表，
	当 Observable 被订阅的时候，OnSubscribe 的 call() 方法会自动被调用.
	事件序列就会依照设定依次触发（对于上面的代码，就是观察者Subscriber 将会被调用三次 onNext() 和一次 onCompleted()）。
	这样，由被观察者调用了观察者的回调方法，就实现了由被观察者向观察者的事件传递，即观察者模式。

创建方式:

1.observable.create(Onsubscriber)
>create() 方法是 RxJava 最基本的创造事件序列的方法。 见上边的代码  使用 create创建的 observable

2.observable.just(T ...)
>基于create()方法， RxJava 还提供了一些方法用来快捷创建事件队列


例如:

	Observable observable = Observable.just("Hello", "Hi", "Aloha");
	// 将会依次调用：
	// onNext("Hello");
	// onNext("Hi");
	// onNext("Aloha");
	// onCompleted();

3.obserbable.from(arraylist<>)
>from(T[]) / from(Iterable<? extends T>) : 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来。

	String[] words = {"Hello", "Hi", "Aloha"};
	Observable observable = Observable.from(words);
	// 将会依次调用：
	// onNext("Hello");
	// onNext("Hi");
	// onNext("Aloha");
	// onCompleted();

上述三个方法本质上是一样的:最终都会运行到:

	public final static <T> Observable<T> create(OnSubscribe<T> f) {
        return new Observable<T>(hook.onCreate(f));
    }
	这个create方法最终执行来创建这个observable;
	


3,进行订阅

	创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：

	订阅方式:(订阅方法中共有5类参数)

	observable.subscribe(observer)
	//or(或者)
	observable.subscribe(subscriber)
	//or(或者)
	observable.subscribe(action1)
	observable.subscribe(action1,action1)
	observable.subscribe(action1,action1,action0)

	他们的本质是一样的 ,subscriber :
	public final Subscription subscribe(Subscriber<? super T> subscriber) {
        return Observable.subscribe(subscriber, this);
    }

	oberver
	public final Subscription subscribe(final Observer<? super T> observer) {
        if (observer instanceof Subscriber) {
            return subscribe((Subscriber<? super T>)observer);
        }
        return subscribe(new Subscriber<T>() {

            @Override
            public void onCompleted() {
                observer.onCompleted();
            }

            @Override
            public void onError(Throwable e) {
                observer.onError(e);
            }

            @Override
            public void onNext(T t) {
                observer.onNext(t);
            }

        });
    }

	也就是说这两个方法本质上都是执行 Observable.subscribe(subscriber, this);的代码的

	PS: public abstract class Subscriber<T> implements Observer<T>, Subscription
	Subscriber 实现了 observer接口 也就是他们其实都是 subseriber;

    
使用action:
1. observable.subscribe(action1)
2. observable.subscribe(action1,action1)
3. observable.subscribe(action1,action1,action0)
4. 还有action3,action4等等.(没有这个,,,,暂时没有, 因为action1代表了next方法和error方法, action0代表了onComplete方法)

至于Action的参数具体为:

	Action1<String> onNextAction = new Action1<String>() {
	    // onNext()
	    @Override
	    public void call(String s) {
	        Log.d(tag, s);
	    }
	};
	Action1<Throwable> onErrorAction = new Action1<Throwable>() {
	    // onError()
	    @Override
	    public void call(Throwable throwable) {
	        // Error handling
	    }
	};
	Action0 onCompletedAction = new Action0() {
	    // onCompleted()
	    @Override
	    public void call() {
	        Log.d(tag, "completed");
	    }
	};

	// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
	observable.subscribe(onNextAction);
	// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
	observable.subscribe(onNextAction, onErrorAction);
	// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
	observable.subscribe(onNextAction, onErrorAction, onCompletedAction);

例子: 
A:打印字符串数组

	String[] names = ...;
	Observable.from(names)
	    .subscribe(new Action1<String>() {
	        @Override
	        public void call(String name) {
	            Log.d(tag, name);
	        }
	    });

B:由 id 取得图片并显示

	由指定的一个 drawable 文件 id drawableRes 取得图片，并显示在 ImageView 中，并在出现异常的时候打印 Toast 报错：

	int drawableRes = ...;
	ImageView imageView = ...;
	Observable.create(new OnSubscribe<Drawable>() {
	    @Override
	    public void call(Subscriber<? super Drawable> subscriber) {
	        Drawable drawable = getTheme().getDrawable(drawableRes));
	        subscriber.onNext(drawable);
	        subscriber.onCompleted();
	    }
	}).subscribe(new Observer<Drawable>() {
	    @Override
	    public void onNext(Drawable drawable) {
	        imageView.setImageDrawable(drawable);
	    }
	
	    @Override
	    public void onCompleted() {
	    }
	
	    @Override
	    public void onError(Throwable e) {
	        Toast.makeText(activity, "Error!", Toast.LENGTH_SHORT).show();
	    }
	});

	正如上面两个例子这样，创建出 Observable 和 Subscriber ，再用 subscribe() 将它们串起来，一次 RxJava 的基本使用就完成了。非常简单。

	在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，如果只用上面的方法，
	实现出来的只是一个同步的观察者模式。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。
	而要实现异步，则需要用到 RxJava 的另一个概念： Scheduler 。

4,线程控制:

1. Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
2. Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
3. Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。
4. Schedulers.computation(): 计算所使用的 Scheduler。
5.  AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。


> 使用方法:使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制。

1. subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。
2. observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

	`int drawableRes = ...;
	ImageView imageView = ...;
	Observable.create(new OnSubscribe<Drawable>() {

	    @Override
	    public void call(Subscriber<? super Drawable> subscriber) {
	
	        Drawable drawable = getTheme().getDrawable(drawableRes));
	        subscriber.onNext(drawable);
	        subscriber.onCompleted();
	    	}
		})

		.subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程

		.observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程

		.subscribe(new Observer<Drawable>() {
		    @Override
		    public void onNext(Drawable drawable) {
		        imageView.setImageDrawable(drawable);
		    }
		 
		    @Override
		    public void onCompleted() {
		    }
		 
		    @Override
		    public void onError(Throwable e) {
		        Toast.makeText(activity, "Error!", Toast.LENGTH_SHORT).show();
		    }
			});`



5,线程变换 :所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。 

1. API : 其本质就是想要把已经获取到的东西进行转换从而变成另一个东西.


	1. map() (一转一);flatMap()(一转到另一个observable);

		`
					
			Observable.just("images/logo.png") // 输入类型 String
	
	    	.map(new Func1<String, Bitmap>() {
	
	        @Override
	        public Bitmap call(String filePath) { // 参数类型 String
	            return getBitmapFromPath(filePath); // 返回类型 Bitmap
	        	}
			})
	
	    	.subscribe(new Action1<Bitmap>() {
	
		        @Override
		        public void call(Bitmap bitmap) { // 参数类型 Bitmap
		            showBitmap(bitmap);
		        }
	    	});`
		区别:
		> flatMap() 和 map() 有一个相同点：它也是把传入的参数转化之后返回另一个对象。但需要注意，和 map()不同的是， flatMap() 中返回的是个 Observable 对象，并且这个 Observable 对象并不是被直接发送到了 Subscriber 的回调方法中。 flatMap() 的原理是这样的：1. 使用传入的事件对象创建一个 Observable 对象；2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable ，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 flatMap() 所谓的 flat。
		
	
	2. throttleFirst(): 在每次事件触发后的一定时间间隔内丢弃新的事件.

		常用作去抖动过滤，例如按钮的点击监听器

			RxView.clickEvents(button) // RxBinding 代码，后面的文章有解释
	
	    		.throttleFirst(500, TimeUnit.MILLISECONDS) // 设置防抖间隔为 500ms
	
	    		.subscribe(subscriber);

			妈妈再也不怕我的用户手抖点开两个重复的界面啦。

	
2. 变换的原理：lift()

	当含有 lift() 时： 

	1.lift() 创建了一个 Observable 后，加上之前的原始 Observable，已经有两个 Observable 了；
 
	2.而同样地，新 Observable 里的新 OnSubscribe 加上之前的原始 Observable 中的原始 OnSubscribe，也就有了两个OnSubscribe； 
	3.当用户调用经过 lift() 后的 Observable 的 subscribe() 的时候，使用的是 lift() 所返回的新的 Observable ，于是它所触发的 onSubscribe.call(subscriber)，也是用的新 Observable 中的新 OnSubscribe，即在 lift() 中生成的那个 OnSubscribe； 

	4.而这个新 OnSubscribe 的 call() 方法中的 onSubscribe ，就是指的原始 Observable 中的原始 OnSubscribe ，在这个 call()方法里，新 OnSubscribe 利用 operator.call(subscriber) 生成了一个新的 Subscriber（Operator 就是在这里，通过自己的call() 方法将新 Subscriber 和原始 Subscriber 进行关联，并插入自己的『变换』代码以实现变换），然后利用这个新Subscriber 向原始 Observable 进行订阅。 

	这样就实现了 lift() 过程，有点像一种代理机制，通过事件拦截和处理实现事件序列的变换。

	eg:  Integer 对象转换成 String

		`
		observable.lift(new Observable.Operator<String, Integer>() {
	    @Override
	    public Subscriber<? super Integer> call(final Subscriber<? super String> subscriber) {
		        // 将事件序列中的 Integer 对象转换为 String 对象
		        return new Subscriber<Integer>() {
		            @Override
		            public void onNext(Integer integer) {
		                subscriber.onNext("" + integer);
		            }
		 
		            @Override
		            public void onCompleted() {
		                subscriber.onCompleted();
		            }
		 
		            @Override
		            public void onError(Throwable e) {
		                subscriber.onError(e);
		            }
		        };
		    }
		});

	讲述 lift() 的原理只是为了更好地了解 RxJava ，从而可以更好地使用它。
	然而不管是否理解了 lift() 的原理，RxJava 都不建议开发者自定义 Operator 来直接使用 lift()，而是建议尽量使用已有的 lift() 包装方法（如 map() flatMap() 等）进行组合来实现需求，
	因为直接使用 lift() 非常容易发生一些难以发现的错误。
		`

3. compose: 对 Observable 整体的变换:除了 lift() 之外， Observable 还有一个变换方法叫做 compose(Transformer).

	它和 lift() 的区别在于:
 
		lift() 是针对事件项和事件序列的，

		而 compose() 是针对 Observable 自身进行变换。

		假设在程序中有多个 Observable ，并且他们都需要应用一组相同的 lift() 变换。

6.线程控制：Scheduler (二)
	
 1. Scheduler 的 API (二)
 
	subscribeOn() 控制事件产生的线程 ... 	一般为背景线程

	observeOn() 控制事件发生的线程 .... 一般在主线程

	如果需要多切换几次线程,使用observeOn()方法,,,,通过 observeOn() 的多次调用，可以程序实现线程的多次切换。

	不同于 observeOn() ， subscribeOn() 的位置放在哪里都可以，因为它是只能调用一次,即只能确定事件产生的线程,所以对位置没有要求.

		Observable.just(1, 2, 3, 4) // IO 线程，由 subscribeOn() 指定
		    .subscribeOn(Schedulers.io())
		    .observeOn(Schedulers.newThread())
		    .map(mapOperator) // 新线程，由 observeOn() 指定
		    .observeOn(Schedulers.io())
		    .map(mapOperator2) // IO 线程，由 observeOn() 指定
		    .observeOn(AndroidSchedulers.mainThread) 
		    .subscribe(subscriber);  // Android 主线程，由 observeOn() 指定

		通过 observeOn() 的多次调用，程序实现了线程的多次切换。

		不过，不同于 observeOn() ， subscribeOn() 的位置放在哪里都可以，但它是只能调用一次的。

 2. Scheduler 的原理（二）
	
	subscribeOn() 和 observeOn() 的内部实现，也是用的 lift()。

	subscribeOn() , 唯一,,最先的线程控制所有线程,,后面的线程在通知过程中就会被最先的subscribeOn()方法截断,,,所以只会是最先的subscribeOn()方法起作用,后面的对整个流程并没有任何影响

	当使用了多个 subscribeOn() 的时候，只有第一个 subscribeOn() 起作用。

 3. doOnSubscribe()  :虽然超过一个的 subscribeOn() 对事件处理的流程没有影响，但在流程之前却是可以利用的。
 
	Subscriber 的 onStart() 可以用作流程开始前的初始化。然而 onStart() 由于在subscribe() 发生时就被调用了，因此不能指定线程，而是只能执行在 subscribe() 被调用时的线程。这就导致如果 onStart() 中含有对线程有要求的代码（例如在界面上显示一个 ProgressBar，这必须在主线程执行），将会有线程非法的风险，因为有时你无法预测subscribe() 将会在什么线程执行。

	而与 Subscriber.onStart() 相对应的，有一个方法 Observable.doOnSubscribe() 。它和 Subscriber.onStart() 同样是在subscribe() 调用后而且在事件发送前执行，但区别在于它可以指定线程。默认情况下， doOnSubscribe() 执行在 subscribe() 发生的线程；而如果在 doOnSubscribe() 之后有 subscribeOn() 的话，它将执行在离它最近的 subscribeOn() 所指定的线程。

	示例代码：

	
	`

		Observable.create(onSubscribe)
		    .subscribeOn(Schedulers.io())
		    .doOnSubscribe(new Action0() {
		        @Override
		        public void call() {
		            progressBar.setVisibility(View.VISIBLE); // 需要在主线程执行
		        }
		    })
		    .subscribeOn(AndroidSchedulers.mainThread()) // 指定主线程
		    .observeOn(AndroidSchedulers.mainThread())
		    .subscribe(subscriber);

	`

	如上，在 doOnSubscribe()的后面跟一个 subscribeOn() ，就能指定准备工作的线程了。

7.RxJava 的适用场景和使用方式

###与 Retrofit 的结合

	@GET("/user")
	public void getUser(@Query("userId") String userId, Callback<User> callback);

	PS: retrifit中 baseurl后面跟  /user : 这里不推荐这样写 ,最好的写法应该是 @GET("user"),这样简单易用不易出问题

	这是接口中的方法,用来回去用户信息的, get请求, query参数代表了要 请求最终为: /user?userId=23  这种形式

在程序的构建过程中， Retrofit 会把自动把方法实现并生成代码，然后开发者就可以利用下面的方法来获取特定用户并处理响应：

	getUser(userId, new Callback<User>() {
	    @Override
	    public void success(User user) {
	        userView.setUser(user);
	    }
	
	    @Override
	    public void failure(RetrofitError error) {
	        // Error handling
	        ...
	    }
	};	
>注 : 这应该是 retrofit 1.x的了, retrofit2 的版本直接会返回call而不能用void方法

而使用 RxJava 形式的 API，定义同样的请求是这样的：

	@GET("/user")
	public Observable<User> getUser(@Query("userId") String userId);
> 这个时候已经获取到 Observable对象了 ,即已经获取到 被观察者了 ,可以进行subscribe进行订阅了

	getUser(userId)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
        @Override
        public void onNext(User user) {
            userView.setUser(user);
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable error) {
            // Error handling
            ...
        }
    });

当 RxJava 形式的时候，Retrofit 把请求封装进 Observable ，在请求结束后调用 onNext() 或在请求失败后调用 onError()。

	你的程序取到的 User 并不应该直接显示，而是需要先与数据库中的数据进行比对和修正后再显示。使用 Callback 方式大概可以这么写
	getUser(userId, new Callback<User>() {
	    @Override
	    public void success(User user) {
	        processUser(user); // 尝试修正 User 数据
	        userView.setUser(user);
	    }
	
	    @Override
	    public void failure(RetrofitError error) {
	        // Error handling
	        ...
	    }
	};
>这是callback的方式, 并不方便因为这样做会影响性能。数据库的操作很重，一次读写操作花费 10~20ms 是很常见的，这样的耗时很容易造成界面的卡顿。所以通常情况下，如果可以的话一定要避免在主线程中处理数据库。所以为了提升性能，这段代码可以优化一下：

	getUser(userId, new Callback<User>() {
	    @Override
	    public void success(User user) {
	        new Thread() {
	            @Override
	            public void run() {
	                processUser(user); // 尝试修正 User 数据
	                runOnUiThread(new Runnable() { // 切回 UI 线程
	                    @Override
	                    public void run() {
	                        userView.setUser(user);
	                    }
	                });
	            }).start();
	    }
	
	    @Override
	    public void failure(RetrofitError error) {
	        // Error handling
	        ...
	    }
	};
>性能问题解决，但……这代码实在是太乱了，迷之缩进啊！杂乱的代码往往不仅仅是美观问题，因为代码越乱往往就越难读懂，而如果项目中充斥着杂乱的代码，无疑会降低代码的可读性，造成团队开发效率的降低和出错率的升高。

如果用 RxJava 的形式，就好办多了。 RxJava 形式的代码是这样的：

	getUser(userId)
	    .doOnNext(new Action1<User>() {
	        @Override
	        public void call(User user) {
	            processUser(user);
	        })
	    .observeOn(AndroidSchedulers.mainThread())
	    .subscribe(new Observer<User>() {
	        @Override
	        public void onNext(User user) {
	            userView.setUser(user);
	        }
	
	        @Override
	        public void onCompleted() {
	        }
	
	        @Override
	        public void onError(Throwable error) {
	            // Error handling
	            ...
	        }
    	});

	后台代码和前台代码全都写在一条链中，明显清晰了很多。

####例子2

假设 /user 接口并不能直接访问，而需要填入一个在线获取的 token ，代码应该怎么写？

 Callback 方式，可以使用嵌套的 Callback：

	@GET("/token")
	public void getToken(Callback<String> callback);
	
	@GET("/user")
	public void getUser(@Query("token") String token, @Query("userId") String userId, Callback<User> callback);

	getToken(new Callback<String>() {
	    @Override
	    public void success(String token) {
	        getUser(token, userId, new Callback<User>() {
	            @Override
	            public void success(User user) {
	                userView.setUser(user);
	            }
	
	            @Override
	            public void failure(RetrofitError error) {
	                // Error handling
	                ...
	            }
	        };
	    }
	
	    @Override
	    public void failure(RetrofitError error) {
	        // Error handling
	        ...
	    }
	});

	倒是没有什么性能问题，可是迷之缩进毁一生，你懂我也懂，做过大项目的人应该更懂。

而使用 RxJava 的话，代码是这样的：

	@GET("/token")
	public Observable<String> getToken();
	
	@GET("/user")
	public Observable<User> getUser(@Query("token") String token, @Query("userId") String userId);
	
	...
	
	getToken()
	    .flatMap(new Func1<String, Observable<User>>() {
	        @Override
	        public Observable<User> onNext(String token) {
	            return getUser(token, userId);
	        })
	    .observeOn(AndroidSchedulers.mainThread())
	    .subscribe(new Observer<User>() {
	        @Override
	        public void onNext(User user) {
	            userView.setUser(user);
	        }
	
	        @Override
	        public void onCompleted() {
	        }
	
	        @Override
	        public void onError(Throwable error) {
	            // Error handling
	            ...
	        }
	    });

	用一个 flatMap() 就搞定了逻辑，依然是一条链


	

	



	
	









