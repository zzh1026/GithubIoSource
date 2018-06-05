---

title: Kotlin笔记
date: 2018-05-16 13:54:08
tags: Android,kotlin
categories: 
    - Kotlin
    - 学习知识

---

# 通过kotlin可以得到什么

1,易表现

通过Kotlin,可以更容易地避免模版代码,因为大部分的典型情况都在语言中默认覆盖实现了。(例如 javabean 类的编写)

<!-- more -->

java中

    `public class Artist {
		 private long id;
		 private String name;
		 public long getId() {
		 	return id;
		 }
		 public void setId(long id) {
		 	this.id = id;
		 }
		 public String getName() {
		 	return name;
		 }
		 public void setName(String name) {
		 	this.name = name;
		 }
		 @Override public String toString() {
			 return "Artist{" +
			 "id=" + id +
			 ", name='" + name + '\'' +
			 '}';
		 }
	}`

kotlin中,我们只需要通过数据类：

	data class Artist(
		var id: Long,
		var name: String,
		var url: String,
		var mbid: String)

这个数据类，它会自动生成所有属性和它们的访问器，以及一些有用的方法，比如， toString()

2,空安全

Java中,如果我们不想遇到 NullPointerException ，我们就需要在使用它之前不停地去判断它是否为null

而kotlin中,还有很多现代的语言，是空安全的,因为我们需要通过一个 安全调用操作符（写做?）来明确地指定一个对象是否能为空。
因此非空调用方法可以使用:  ?. 方法, 表示非空的时候调用方法, 如果为空的时候执行某方法的化 使用  ?:let{}	,即?表示空判断 .表示非空调用方法 :表示为空的时候执行 后面接 let{} 方法体就执行let方法中的内容.

	// 这里不能通过编译. Artist 不能是null
	var notNullArtist: Artist = null
	
	// Artist 可以是 null , 即如果一个数据可能为空时需要声明的,默认没有空值, 而java中默认是空
	var artist: Artist? = null

	// 无法编译, artist可能是null，我们需要进行处理
	artist.print()  可以通过 artist?.print()

	// 智能转换. 如果我们在之前进行了空检查，则不需要使用安全调用操作符调用	类似的还有 as :as 和java中的instenceof 相似,但是使用as后如果为true 就会自动转换为 as 之后的对象了
	if (artist != null) {
		artist.print()
	}

	// 只有在确保artist不是null的情况下才能这么调用，否则它会抛出异常
	artist!!.print()	//!! 应该尽量少用,可以使用?.

	// 使用Elvis操作符来给定一个在是null的情况下的替代值
	val name = artist?.name?: "empty"


3,扩展方法

我们可以给任何类添加函数。它比那些我们项目中典型的工具类更加具有可读性。这样做的优点是 不需要为了给一些共有函数的共有方法抽取父类来进行统一添加某个方法, 可以直接在其父类中扩展一个 方法来使得其子类都含有该内容

我们可以给fragment增加一个显示toast的函数：

	fun Fragment.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
		Toast.makeText(getActivity(), message, duration).show()
	}

这样当我们在任何地方有该fragment或者其子类的时候都可以调用

	fragment.toast("Hello world!")

4,函数式支持(Lambdas) 将函数当做参数传入

	有如下规则: 函数表达式的基本形式是 (参数) -> 返回值
	所以传入的方法是 : view.setOnClickListener(listener(View) -> { toast("Hello world!") })  //参数为 View , 结果为 toast()
	由于 当参数没有用到的时候 可以省略参数 

	每次我们去声明一个点击所触发的事件，可以只需要定义我们需要做些什么，而不是不得不去实现一个内部类？我们确实可以这么做，这个我们需要感谢lambda： 本质上就是 匿名内部类的 简写
	{ new OnclickListener(){} 省略 然后直接写其中的方法}

	`view.setOnClickListener { toast("Hello world!") }`

即我们可以省略内部类的运用 具体调用方式为:当需要传入一个内部类作为参数的时候 可以 使用直接 {} 的方式代替, 然后{}中直接写入需要实现的方法体即可.

##app

###1,类的继承

当一个类继承父类的时候, 如果父类是一个类, 则继承的时候需要实现super方法,就像这样:

	class MainActivity : AppCompatActivity(){}

但是如果父类是一个接口 ,则实现(继承)的时候可以不体现出super方法来 ,就像这样

	class MainActivity : OnClickListener{}

默认任何类都是基础继承自 Any  （与java中的 Object  类似），但是我们可以继承其它类。所有的类默认都是不可继承的（final），所以我们只能继承那些明确声明 open  或者 abstract  的类:

	open class Animal(name: String)

	class Person(name: String, surname: String) : Animal(name)

当我们只有单个构造器时，我们需要在从父类继承下来的构造器中指定需要的参数。这是用来替换Java中的 super  调用的。

###2,定义一个类

kotlin中 表示一个类 的关键字是 class。

	class MainActivity{}

它有一个默认唯一的构造器。如果需要参数只需要在类名后面写上它的参数。如果这个类没有任何内容可以省略大括号：

	class Person(name: String, surname: String)	

kotlin中没有static关键字, 也就没有静态代码块的说法, 所以如果想要在调用类之前初始化数据 需要使用 init{} 方法,即

	class Person(name: String, surname: String) {
		init{
			//...
		}
	}

###3,函数

函数可以使用 fun 关键字来定义:

	fun onCreate(savedInstanceState: Bundle?) {
	}

如果你没有指定它的返回值，它就会返回 Unit，与Java中的 void  类似，但是 Unit 是一个真正的对象。也可以指定任何其它的返回类型：

	fun add(x: Int, y: Int) : Int {
		return x + y
	}
    
> kotlin中每行代码后面的分号不是必须的

然而如果返回的结果可以使用一个表达式计算出来，你可以不使用括号而是使用等号： 这里的重点是可以使用一个表达式 才可以使用等号

	fun add(x: Int,y: Int) : Int = x + y

###4,构造方法和函数参数

Kotlin中的参数与Java中有些不同.我们需要先写参数的名字再写它的类型:	

	fun add(x: Int, y: Int) : Int {
		return x + y
	}

我们可以给参数指定一个默认值使得它们变得可选，这是非常有帮助的。这里有一个例子，在Activity中创建了一个函数用来toast一段信息：

	fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
		Toast.makeText(this, message, length).show()
	}

	fun niceToast(message: String,
	tag: String = javaClass<MainActivity>().getSimpl
	eName(),
	length: Int = Toast.LENGTH_SHORT) {
		Toast.makeText(this, "[$tag] $message", length).show()
	}

> String模版
> 
> 在String中直接使用模版表达式。它可以帮助你很简单地在静态值和
> 变量的基础上编写复杂的String。在kotlin中任何时候使用一个 $  符号就可以插入一个表达式。如果这个表达式有一点复杂，则需要使用一对大括号括起来："Your name is${user.name}"。

##1,主界面

####对象实例化

>对象实例化也是与Java中有些不同。kotlin中去掉了 new 关键字。这时构造函数仍然会被调用，但是省略了宝贵的四个字符.LinearLayoutManager(this)  创建了一个对象的实例.

getter和setter的使用

>当类中有setter 和getter方法的时候 ,在kotlin中来调用很简单,例如 在recycler 中:

>recyclerView.layoutManager 或者 recyclerView.adapter表示 getter方法, 即他们表示 结果

>recyclerView.layoutManager = LinearLayoutManager(this) 和 recyclerView.adapter = WeatherAdapter(this, items) 则表示setter 方法 表示设置,结合对象的实例化则可以设置和获取了.

list集合的创建

>可以通过使用一个函数 listOf  创建一个常量的List。它接收一个任何类型的 vararg  （可变长的参数），它会自动推断出结果的类型。

>还有很多其它的函数可以选择，比如 setOf  ， arrayListOf  或者 hashSetOf  。

####变量和属性

在Kotlin中，一切都是对象。没有像Java中那样的原始基本类型。这个是非常有帮助的，因为我们可以使用一致的方式来处理所有的可用的类型。

####基本类型

kotlin中像integer，float或者boolean等类型仍然存在，但是它们全部都会作为对象存在的。基本类型的名字和它们工作方式都是与Java非常相似的，但是有一些不同之处：

1,数字类型中不会自动转型: 不能给 Double  变量分配一个 Int  。必须要做一个明确的类型转换，可以使用众多的函数之一：

	val i:Int=7
	val d: Double = i.toDouble()

2,字符（Char）不能直接作为一个数字来处理。在需要时我们需要把他们转换为一个数字：

	val c:Char='c'
	val i: Int = c.toInt()

3,位运算也有一点不同。在Android中，我们经常在 flags  中使用“或”，所以我使用"and"和"or来举例：

	// Java
	int bitwiseOr = FLAG1 | FLAG2;
	int bitwiseAnd = FLAG1 & FLAG2;

	// Kotlin
	val bitwiseOr = FLAG1 or FLAG2
	val bitwiseAnd = FLAG1 and FLAG2

4,字面上可以写明具体的类型。这个不是必须的，但是一个通用的Kotlin实践时省略变量的类型,所以我们可以让编译器自己去推断出具体的类型。

	val i = 12 	//int直接写
	val iHex = 0x0f //一个十六进制的Int类型 十六进制用 0x表示
	val l = 3L // A Long long类型在末位 加上 L 大写字母表示
	val d = 3.5 // int 和double 都可以自动识别
	val f = 3.5F // float 是在 double的基础上 加上大写的F表示

string也可以自动识别

5,一个String可以像数组那样访问，并且被迭代：
	val s = "Example"
	val c = s[2] // 这是一个字符'a'
	// 迭代String
	val s = "Example"
	for(c in s){
		print(c)
	}

####变量

kotlin中 变量可以很简单地定义成可变( var  )和不可变（ val  ）的变量。这个与Java中使用的 final  很相似。但是不可变在Kotlin（和其它很多现代语言）中是一个很重要的概念。

kotlin中一个类的声明默认是不可变的 ,如果可以继承必须显式的标识为 abstract 或者 open 的, 但是java中默认是 可以修改或者继承的, 如果不可变是需要 用final修饰的

>一个不可变对象意味着它在实例化之后就不能再去改变它的状态了。如果你需要一个这个对象修改之后的版本，那就会再创建一个新的对象。这个让编程更加具有健壮性和预估性。在Java中，大部分的对象是可变的，那就意味着任何可以访问它这个对象的代码都可以去修改它，从而影响整个程序的其它地方。

>不可变对象也可以说是线程安全的，因为它们无法去改变，也不需要去定义访问控法去改变，也不需要去定义访问控

我们通常不需要去指明类的类型，它会自动从后面分配的语句中推断出来，这样可以让代码更加清晰和快速修改。例如下面的例子:

	val s = "Example" // A String
	val i = 23 // An Int
	val actionBar = supportActionBar // An ActionBar in an Activity context

如果我们需要使用更多的范型类型，则需要指定：

	val a: Any = 23		//因为默认只有一种, 如果想要用其他标识的化则需要声明类型,不过一般用不到, 因为声明的类型一般是其父类型.
	val c: Context = activity

####属性

kotlin中属性与Java中的字段是相同的，但是更加强大。属性做的事情是字段加上getter加上setter。 java中字段安全访问和修改需要getter和setter方法来处理, 而在kotlin中,只需要一个属性就可以了

	public class Person {
		var name: String = ""
	}
	
	val person = Person()
	person.name = "name"	//这个代表set方法
	val name = person.name	//这个代表get方法

如果没有任何指定，属性会默认使用getter和setter。当然它也可以修改为你自定义的代码，并且不修改存在的代码：

	public classs Person {
		var name: String = ""
			get() = field.toUpperCase()
			set(value){
			field = "Name: $value"
		}
	}
可以使用 field  这个预留字段来访问，它会被编译器找到正在使用的并自动创建。

####Anko
Anko是JetBrains开发的一个强大的库。它主要的目的是用来替代以前XML的方式来使用代码生成UI布局。

####扩展函数

扩展函数数是指在一个类上增加一种新的行为，甚至我们没有这个类代码的访问权限。这是一个在缺少有用函数的类上扩展的方法。Kotlin中扩展函数的一个优势是我们不需要在调用方法的时候把整个对象当作参数传入。扩展函数表现得就像是属于这个类的一样，而且我们可以使用 this  关键字和调用所有public方法。 

>扩展函数的具体写法为: fun XXX(//要扩展的类).yyy(//要扩展的方法)(a:Any,b:Any)(//方法的参数){}(//方法体(方法体中的 this 关键字代表 该类))
>
>同时 ,这个函数的位置是属于方法级别的或者包级别的 ,不是在方法内部的.而且可以写在任何地方,所以可以抽取出来单独将同一类函数放在文件中

例如:我们可以创建一个toast函数，这个函数不需要传入任何context，它可以被任何Context或者它的子类调用，比如Activity或者Service：

	fun Context.toast(message:String, duration:Int = Toast.LENGTH_SHORT){
		Toast.makeText(this,message,duration).show()
	}
这个方法可以在Activity内部直接调用：

	toast("Hello world!")
	toast("Hello world!", Toast.LENGTH_LONG)

Anko已经包括了自己的toast扩展函数，跟上面这个很相似。Anko提供了一些针对 CharSequence  和 resource  的函数，还有两个不同的toast和longToast方法：

	toast("Hello world!")
	longToast(R.id.hello_world)

扩展函数也可以是一个属性。所以我们可以通过相似的方法来扩展属性。

>anko包本质上是一个工具类, org.jetbrains.anko:anko-commons 包中主要是对类的扩展方法, 包括不限于: AlertDialog 的封装扩展, Context 的封装扩展 , Logging 的封装扩展, Toast的扩展等等

扩展函数并不是真正地修改了原来的类，它是以静态导入的方式来实现的。扩展函数可以被声明在任何文件中，因此有个通用的实践是把一系列有关的函数放在一个新建的文件里。

>即针对不同扩展可以进行抽取,因为可以在任何文件中声明,而且声明后可以在任何地方即时的使用.

####执行一个请求
>使用网址: [open weather](http://openweathermap.org/)

>一个简单的读取 某个 url 的返回值  
      
>     val forecastJsonStr = URL(url).readText()
    Log.i("alog", forecastJsonStr)

>Url(url).readText() 是kotlin中的保准扩展库,这个方法不推荐结果很大的json。相比较与 java 节省了大量的代码。比如 HttpURLConnection  、 BufferedReader  和需要达到相同效果所必要的迭代结果，管理连接状态、reader等部分的代码。

####在主线程以外执行请求

网络请求不允许在主线程中执行,Android中一般使用AsyncTask , 但是这个类使用效果比较差, Anko提供了非常简单的DSL来处理异步任务，它满足大部分的需求。
>Anko 中的 async  函数用于在其它线程执行代码，也可以选择通过调用 uiThread  的方式回到主线程。在子线程中执行请求如下这么简单：

	doAsync() {
		Request(url).run()
		uiThread { longToast("Request performed") }
	}
>uiThread  有一个很不错的一点就是可以依赖于调用者。如果它是被一个 Activity  调用的，那么如果 activity.isFinishing()  返回 true  ，则 uiThread  不会执行，这样就不会在Activity销毁的时候遇到崩溃的情况了。


##数据类

数据类是一种非常强大的类，它可以让你避免创建Java中的用于保存状态但又操作非常简单的POJO的模版代码。它们通常只提供了用于访问它们属性的简单的getter和setter。定义一个新的数据类非常简单：

	data class Forecast(val date: Date, val temperature: Float, val details: String)

####额外的函数

通过数据类，我们可以方便地得到很多有趣的函数，一部分是来自属性.

>equals(): 它可以比较两个对象的属性来确保他们是相同的。(kotlin 的equals 比较的是对象的属性, 而java中比较的是对象的地址)
>hashCode(): 我们可以得到一个hash值，也是从属性中计算出来的。
>copy(): 你可以拷贝一个对象，可以根据你的需要去修改里面的属性。
>一系列可以映射对象到变量中的函数。

####复制一个数据类

可以使用copy 的方法

	val f1 = Forecast(Date(), 27.5f, "Shiny day")
	val f2 = f1.copy(temperature = 30f)

####映射对象到变量中

映射对象的每一个属性到一个变量中，这个过程就是我们知道的多声明。

	val f1 = Forecast(Date(), 27.5f, "Shiny day")
	val (date, temperature, details) = f1

上面这个多声明会被编译成下面的代码：

	val date = f1.component1()
	val temperature = f1.component2()
	val details = f1.component3()

这个特性背后的逻辑是非常强大的，它可以在很多情况下帮助我们简化代码。举个例子， Map  类含有一些扩展函数的实现，允许它在迭代时使用key和value：

	for ((key, value) in map) {
		Log.d("map", "key:$key, value:$value")
	}

####转换json到数据类

使用 GsonToKotlin 的 plugs 来进行对gson 的转换

>伴随对象 Companion objects
>
>>Kotlin允许我们去定义一些行为与静态对象一样的对象。尽管这些对象可以用众所周知的模式来实现，比如容易实现的单例模式。
>>
>>我们需要一个类里面有一些静态的属性、常量或者函数，我们可以使用 companion objecvt  。这个对象被这个类的所有对象所共享，就像Java中的静态属性或者方法。
>>
>
>这是因为 kotln 中没有 statc 的关键字, 所以想要使用静态常量就必须伴随对象了

伴随对象的写法:

	public class ForecastRequest(val zipCode: String="1816670") {
		companion object {
			private val URL = "http://samples.openweathermap.org/data/2.5/forecast/daily?appid=b1b15e88fa797225412429c1c50c122a1&id="
			private val COMPLETE_URL = "$URL&id="
		}

		fun execute(): ForecastResult {
			val forecastJsonStr = URL(COMPLETE_URL + zipCode).readText()
			return Gson().fromJson(forecastJsonStr, ForecastResult::class.java)
		}
	}

####构建domain层
我们现在创建一个新的包作为 domain  层。这一层中会包含一些 Commands  的实现来为app执行任务。

首先，必须要定义一个 Command  ：

	public interface Command<T> {
		fun execute(): T
	}

这个command会执行一个操作并且返回某种类型的对象，这个类型可以通过范型被指定。
>一切kotlin函数都会返回一个值。如果没有指定，它就默认返回一个 Unit  类。
>所以如果我们想让Command不返回数据，我们可以指定它的类型为Unit。
> 1 ,如果当 返回值可以句代码来处理可以直接使用 = 结果操作 的方式来快速返回
> 2 , Kotlin 中的接口 中声明的方法可以包含代码
> 3 , 当我们使用了两个相同名字的类，我们可以给其中一个指定一个别名，这样我们就不需要写完整的包名了：
> >import com.czb.weatherkotlin.bean.Forecast as ModelForecast

#####知识点: 在创建 list 的时候 可以使用方法 

	list.map { convertForecastItemToDomain(it) }
 
> 这个 map 方法是 kotlin 中对 collectons 类的标准扩展库, 用来将一个 可迭代的 数据转换成一个 list<R> 数据,其源码是:

    public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    	return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
	}

>通过这一条语句，我们就可以循环这个集合并且返回一个转换后的新的List。
>Kotlin在List中扩展提供了很多不错的函数操作符，它们可以在这个List的每个item中应用这个操作并且任何方式转换它们。
>it 指代的是 list 中迭代的每一个数据

adapter 中:

	class WeatherAdapter(val weekForecast: ForecastList) : RecyclerView.Adapter<WeatherAdapter.WeatherViewHolder>() {

    	override fun getItemCount(): Int {
	        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
	    }
	
	    override fun onBindViewHolder(holder: WeatherViewHolder, position: Int) {
	        with(weekForecast.forecast[position]) {
	            holder.itemView as TextView
	            holder.itemView.text = "$date - $notice - $high - $low"
	        }
	    }
	
	    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): WeatherViewHolder {
	        return WeatherViewHolder(TextView(parent.context))
	    }
	
	    class WeatherViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
	    }
	}

>with函数  
>>with是一个非常有用的函数，它包含在Kotlin的标准库的util工具包中。
>>它接收一个对象和一个扩展函数作为它的参数，然后使这个对象扩展这个函数。
>>这表示所有我们在括号中编写的代码都是作为对象（第一个参数）的一个扩展函数，我们可以就像作为this一样使用所有它的public方法和属性。
>>当我们针对同一个对象做很多操作的时候这个非常有利于简化代码。

with函数的使用方法是:  with(扩展对象){扩展函数}

	with(object){
		//该方法体内可以直接使用object的方法, 就相当于在 object内部一样来操作其中的数据
	}

##操作符重载

Kotin有一些固定数量象征性的操作符，我们可以在任何类中很容易地使用它们。
>创建一个方法，方法名为保留的操作符关键字，这样就可以让这个操作符的行为映射到这个方法。重载这些操作符可以增加代码可读性和简洁性。

####操作符表

 操作符  和 对应方法 对应方法必须在指定的类中通过各种可能性被实现。

>操作符是运用在代码中的运算方式, java中直接对操作符进行操作, 而在kotlin中会对操作符进行统一,即将进行对象进行 方法计算

一元操作符

>+a --> a.unaryPlus()
>
>-a --> a.unaryMinus()
>
>!a --> a.not()
>
>a++ --> a.inc()
>
>a-- --> a.dec()

二元操作符

>a+b  -->  a.plus(b)
>
>a-b  -->  a.minus(b)
>
>a*b  -->  a.times(b)
>
>a/b  -->  a.div(b)
>
>a%b  -->  a.mod(b)
>
>a..b -->  a.rangeTo(b)   --  表示从 a到 b(1..4表示 从1到4)
>
>a in b  -->  a.contans(b) -- 表示a是否在b中
>
>a !in b -->  !a.contains(b)
>
>a+=b -->  a.plusAssign(b)  --  表示a+b的结果赋值给a
>
>a-=b -->  a.minusAssign(b)
>
>a*=b -->  a.tiimesAssign(b)
>
>a/=b -->  a.divAssign(b)
>
>a%=b -->  a.modAssign(b)

数组操作符

>a[i]  -->  a.get(i)
>
>a[i,j] --> a.get(i,j)
>
>a[i_1,...,i_n] --> a.get(i_1,...,i_n)
>
>a[i]=b --> a.set(i,b)
>
>a[i,j]=b -->  a.set(i,j,b)
>
>a[i_q,...,i_n]=b  -->  a.set(i_1,...,i_n,b)

等于操作符

>a==b --> a?.equels(b)?:b===null -- 表示a若不为null和b进行比较
>
>a!=b --> !(a?.equals(b)?:b===null)
>
>>相等操作符有一点不同，为了达到正确合适的相等检查做了更复杂的转换
>
>> kotlin中的 === 符号相当于 java中的 == ,!== 相当于 !=

函数调用

>a(i)  -->  a.invoke(i)
>
>a(i,j) --> a.invoke(i,j)
>
>a(i_1,...,i_n)  -->  a.invoke(i_1,...,i_n)

>kotln list是实现了 数字操作符的,所以可以像java中的数组一样访问list的每一项.

####扩展函数中的操作符 -->  将操作符灵活运用(因为kotlin中的每一个操作符 都是有其对应的方法,扩展这些方法就可以实现更加方便的功能)

>如果要扩展操作符 , 则要扩展的方法 必须用 operator 关键字修饰
>
>操作符的本质就是 通过操作符可以用来替换 其代表的方法 , 所以当 想要使用  a[i] 的时候 则代表 调用 a.get(i) 的方法, 因此 a中必须要有 get(position:Int) 的方法. 同时由于想要使用 操作符,则 get方法必须用 operator 修饰, 否则只能使用 a.get(i)方法 而不能使用操作符 a[i], 因此operator 本质上表示声明一下该方法可以被使用作操作符
>List 中可以使用 a[i] 来方便快捷的获取到i位置的 对象就是因为 List集合被 kotlin 扩展了 get() 方法, 同时该方法用 operator 来修饰了.

我们不需要去扩展我们自己的类，但是我需要去使用扩展函数扩展我们已经存在的类来让第三方的库能提供更多的操作。

例如:我们可以去像访问List的方式去访问 ViewGroup  的view：

	operator fun ViewGroup.get(position: Int): View = getChildAt(position)

在代码中

	val container: ViewGroup = find(R.id.container)
	val view = container[2]

>对类的扩展可以分为两种, 扩展方法和扩展属性, 扩展属性的化需要为其设置 get()={} ,的获取方法

扩展 Context 的一个属性 ctx

	val View.ctx: Context
	    get() = context

####kotlin中创建匿名内部类

	recyclerView.adapter = WeatherAdapter(result, object : OnItemClickListener {
	                    override fun invoke(forecast: Forecast) {
	                        toast("设置成功")
	                    }
	                })

创建一个匿名内部类，我们去创建了一个实现了刚刚创建的接口的对象。


##Lambdas

####Lambda表达式是一种很简单的方法，去定义一个匿名函数。Lambda是非常有用的，因为它们避免我们去写一些包含了某些函数的抽象类或者接口，然后在类中去实现它们。在Kotlin中，我们把一个函数作为另一个函数的参数。

>lambdas表达式的主要功能有
>> 1,避免抽象类的接口, 可以使代码更加简洁  a.setOnclickListener{toast("aa")}

####lambdas 使用 1 ,简化setOnClickListener()

在java中 , 首先要编写一个 OnClickListener  接口,然后要编写一个匿名内部类去实现这个接口. 当使用匿名内部类的时候 , 需要使用 object 来声明匿名,, 因为如果使用其他名字就不是匿名了

将java函数转换成kotlin,也是使用了 内部类的方式

	view.setOnClickListener(object : OnClickListener {
		override fun onClick(v: View) {
			toast("Click")
		}
	}

Kotlin允许Java库的一些优化

当Interface中包含单个函数可以被替代为一个函数。因此,上面的代码会正常执行这段代码:

	fun setOnClickListener(listener: (View) -> Unit)
	
	这样表示 setOnclickListener 方法中接收一个 Listener , 参数为 View,返回值Unit	

一个lambda表达式通过参数的形式被定义在箭头的左边（被圆括号包围），然后在箭头的右边返回结果值。在这个例子中，我们接收一个View，然后返回一个Unit（没有东西）。所以根据这种思想，我们可以把前面的代码简化成这样：

	view.setOnClickListener({ view -> toast("Click")})

当我们定义了一个方法，我们必须使用大括号包围，然后在箭头的左边指定参数，在箭头的右边返回函数执行的结果。##### 如果左边的参数没有使用到，我们甚至可以省略左边的参数： 

	view.setOnClickListener({ toast("Click") })

如果这个函数的#### 最后一个 ####参数是一个函数，我们可以把这个函数移动到圆括号外面：最后一个参数是函数可以移出来, 意味着如果一个函数移出来了说明该函数是最后一个参数

	view.setOnClickListener() { toast("Click") }

并且，最后，如果这个函数没有其他的参数，我们可以省略这个圆括号：

	view.setOnClickListener { toast("Click") }

比原始的Java代码简短了5倍多，并且更加容易理解它所做的事情。非常让人影响深刻。

####lambdas 使用 2 , 简化其他代码
因此,在实际使用的时候 WeatherAdapter 接收的参数变成了
	WeatherAdapter(val weekForecast: ForecastList, val itemClick: (Forecast) -> Unit) 

>上面的代码表示 adapter中接收一个 名字为 itemClick 的参数 , 该参数是 lambdas 表达式 ,表示 接收一个 Forecast 对象的函数 ,返回一个 Unit ,因此 ,adapter就成了

	val adapter = WeatherAdapter(result, { forecast -> toast("aa") })

>由于最后一个参数是一个函数 ,所以将函数移动到括号外面

	val adapter = WeatherAdapter(result) { forecast -> toast("aa") }

>因为还有其他的参数 ,不能省略括号, 参数没有用到 所以最终的代码为

	val adapter = WeatherAdapter(result) { toast("aa") }
>同时, 如果一个函数只接受一个参数, 那我们可以使用 it  引用，而不用去指定左边的参数。因此 ,adapter 的实际代码为:

	val adapter = ForecastListAdapter(result) { toast(it.date) }

####lambdas 使用 3 , 扩展语言

我们可以去创建自己的 builder  和代码块。我们已经在使用一些有趣的函数，比如 with  。如下简单的实现：
	inline fun <T> with(t: T, body: T.() -> Unit) { t.body() }

>这里的第二个参数  body: T.() -> Unit 也是一个lambdas表达式

>方法是with ,接受两个参数 t:T ,表示一个对象; body:T.() ,表示接收一个函数,这个函数的方法名为body ; -> Unit ,表示无返回值 ;{t.body()} ,这是 with方法的方法体, 和调用with方法的对象无关, 这个方法体表示 对象 t 调用传入的方法 body() ,因此, with 在使用的时候一般是:()

	with(object,{})
因为没有返回值,或者第二个参数是一个函数,所以将内容放在括号外面
	with(object){
	}

这个函数接收一个 T  类型的对象和一个被作为扩展函数的函数。它的实现仅仅是让这个对象去执行这个函数。因为第二个参数(最后一个参数)是一个函数，所以我们可以把它放在圆括号外面，所以我们可以创建一个代码块，在这这个代码块中我们可以使用 this  和直接访问所有的public的方法和属性：
	with(forecast) {
		Picasso.with(itemView.ctx).load(iconUrl).into(iconView)
		dateView.text = date
		descriptionView.text = description
		maxTemperatureView.text = "$high"
		minTemperatureView.text = "$low"
		itemView.setOnClickListener { itemClick(this) }
	}

>内联函数
>>内联函数与普通的函数有点不同。一个内联函数会在编译的时候被替换掉，而不是真正的方法调用。
>这在一些情况下可以减少内存分配和运行时开销。
>例如:如果我们有一个函数，只接收一个函数作为它的参数。如果是一个普通的函数，内部会创建一个含有那个函数的对象。
>>另一方面，内联函数会把我们调用这个函数的地方替换掉，所以它不需要为此生成一个内部的对象。

例如: 我们可以创建代码块只提供 Lollipop  或者更高版本来执行：

	inlne fun supportsLollipop(code:()->Unit){
		if(Build.VERSION.SDK_INT >= Build.VERSIION_CODES.LOLLIIPOP){
			code()
		}
	}

它只是检查版本，然后如果满足条件则去执行。现在我们可以这么做：

	supportsLollipop {
		window.setStatusBarColor(Color.BLACK)
	}

Anko也是基于这个思想来实现 Android Layout  的 DSL  化。

##可见性修饰符

Kotlin中的修饰符是与我们Java中的使用有些不同的。在这个语言中默认的修饰符是 public  ，这节约了很多的时间和字符。因为 默认是 public的 ,所以在其他地方默认的参数都是可以使用的, java中默认是 default 的,它的权限和 protected 差不多, 所以其他地方默认不可使用

###修饰符

####private -- 可以修饰 类,接口,成员 

private  修饰符是我们使用的最限制的修饰符。它表示它只能被自己所在的文件可见,如果我们给一个类声明为 private , 我们就不能再定义这个类之外的文件中使用它.

另一方面,如果我们在一个类里面使用了 private 修饰符,那访问权限就被限制在这个类里面了.甚至是继承这个类的子类也不能使用它.

所以一等公民，类、对象、接口……（也就是包成员）如果被定义为 private  ，那么它们只会对被定义所在的文件可见。如果被定义在了类或者接口中，那它们只对这个类或者接口可见。

####protected -- 可以修饰 成员

这个修饰符只能被用在类或者接口中的成员上。一个包成员不能被定义为 protected  。

定义在一个成员中，就与Java中的方式一样了：它可以被成员自己和继承它的成员可见（比如，类和它的子类）。

####internal -- 可以修饰 类,接口,成员 

如果是一个定义为 internal  的包成员的话，对所在的整个 module  可见。
>如果是 类或者接口 对module可见

如果它是一个 成员，它就需要依赖那个领域的可见性了。
>比如，如果我们写了一个 private  类，那么它的 internal  修饰的函数的可见性就会限制与它所在的这个类的可见性。
>成员用 internal 修饰的时候理论上也是可以被 module 中访问, 但是由于类可能被 private 修饰了, 造成了其他地方不可以访问了

我们可以访问同一个 module  中的 internal  修饰的类，但是不能访问其它 module  的。

>什么是 module
>>根据Jetbrains的定义，一个 module  应该是一个单独的功能性的单位，它应该是可以被单独编译、运行、测试、debug的。
>>根据我们项目不同的模块，可以在Android Studio中创建不同的 module。
>>即一个module就是一个模块, 实际项目中 app 属于一个模块,当依赖第三方库的时候,第三方库就是一个module

####public -- 可以修饰 类,接口,成员

最没有限制的修饰符。默认的修饰符，成员在任何地方被修饰为 public  ，很明显它只限制于它的领域。

一个定义为 public  的成员被包含在一个 private  修饰的类中，这个成员在这个类以外也是不可见的。

>public默认修饰类的, 所以这个类可以轻易被找到; 而java中默认不是public ,所以没有被public修饰的类不能访问.
>kotlin中 的类不仅默认被public修饰, 而且默认被 final 修饰,final 和 open,protected 是互斥的 ,被final修饰的类不能被继承,所以kotlin中需要被继承的类是需要用 open 或者 protected 修饰的, 而java中 默认没有被final修饰, 所以是默认可以继承的. 
>即 public修饰 可见性, open 修饰可否被继承, protected修饰的类 默认是 open的,因为 protected如果默认被 final修饰的话没意义.

###构造器 -- 构造函数的多种样式

所有构造函数默认都是 public  的，它们类是可见的，可以被其它地方使用。我们也可以使用这个语法来把构造函数修改为 private  ：在 类声明后 添加 private constructor

	class C ( a: Int ){ ... } 

	class C private constructor(a: Int) { ... }

在Kotlin中，我们不需要去指定一个函数的返回值类型，它可以让编译器推断出来。例如:

	data class ForecastList(val city: String, val date: String,
	                        private val forecast: List<Forecast>) {
	    operator fun get(position: Int) = forecast[position]
	    fun size(): Int = forecast?.size
	}
>这里的 operator fun get(position: Int) = forecast[position] 方法就非常好用
>首先 operator+get(position)方法可以使用 obj[position] 来获取
>其次 forecast[position] 获取到的是一个非常明确的 Forecast 对象,因此当get方法返回的是一个 Forecast 对象的时候就可以不指定返回值类型 进而让编译器自动推断出来返回结果.

我们可以省略返回值类型的典型情景是当我们要给一个函数或者一个属性赋值的时候。而不需要去写代码块去实现。 即当调用  =xxx 方法的时候因为赋值的时候对象很明确,所以可以不需要声明返回值类型.如果有代码块的时候一般的返回值是不确定的.

##Kotlin Android Extensions -- 类似于databinding 的 kotlin插件

使用方式: 

 在activity中需要手动导入activity 的 layout 

	import kotlinx.android.synthetic.main.activity_main.*

 如果actvity 中 通过 include 标签增加了 内嵌的布局, 还需要手工导入这个layout
	
	import kotlinx.android.synthetic.main.content_main.*

 如果在 adapter 或者 自定义 view 中,也需要 导入这个 layout

	import kotlinx.android.synthetic.main.item_forecast.view.*

 如果我们需要一个adapter，比如，我们现在要从inflater的View中访问属性：

	view.textView.text = "Hello"

>新版的中 ,最简单的方法就是直接使用, 然后会自动导入, 添加上classespath 之后就可以直接使用了.

##Application单例化和属性的Delegated

### Application单例化 ###

原因:Android有一个问题，就是我们不能去控制很多类的构造函数。比如，我们不能初始化一个非null属性，因为它的值需要在构造函数中去定义。所以我们需要一个可null的变量，和一个返回非null值的函数。我们知道我们一直都有一个 App  实例，但是在它调用 onCreate  之前我们不能去操作任何事情，所以我们为了安全性，我们假设 instance()  函数将会总是返回一个非null的 app  实例。

但是这个方案看起来有点不自然。我们需要定义个一个属性，然后通过一个函数来返回那个属性。因此,我们可以通过委托这个属性的值给另外一个类。这个就是我们知道的 委托属性  。

### 委托属性 ###

我们可能需要一个属性具有一些相同的行为，使用 lazy  或者 observable  可以被很有趣地实现重用。而不是一次又一次地去声明那些相同的代码，Kotlin提供了一个委托属性到一个类的方法。这就是 委托属性  。

>委托属性 指的是 将 一个类的属性的值 委托到另一个类, 在一个单例类中, 就可以将自己的单例的 instence 属性委托到另一个类 , 这样当其他的 类中需要单例的时候也可以委托 其instence 到这个类中, 就避免重复的每个类都为自己的 单例 来赋值,这样就不需要声明 单例创建的那些相同的代码了

当我们使用属性的 get  或者 set  的时候，属性委托 的 getValue  和 setValue  就会被调用。

属性委托的结构如下：

	class Delegate<T> : ReadWriteProperty<Any?, T> {
		fun getValue(thisRef: Any?, property: KProperty<*>): T {
			return ...
		}
		fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
			...
		}
	}

这个T是委托属性的类型. getValue 函数接收一个类的引用和一个属性的 元数据.
setValue函数又接收了一个 被设置的值. 如果这个属性是不可修改(val),就只会有一个getValue函数

属性委托的设置方法：

	class Example {
		var p: String by Delegate()
	}

使用了 by  这个关键字来指定一个委托对象。
>委托的 关键字就是 by关键字, 通过by可以指定委托对象.

### 标准委托 ###

Kotlin的标准库中有一系列的标准委托。它们包括了大部分有用的委托，但是我们也可以创建我们自己的委托。

#### Lazy ####
它包含一个lambda，当第一次执行 getValue  的时候这个lambda会被调用，所以这个属性可以被延迟初始化。之后的调用都只会返回同一个值。当我们在它们第一次真正调用之前不是必须需要它们的时候。我们可以节省内存，在这些属性真正需要前不进行初始化。

>延迟初始化, 即在首次调用之前不进行初始化, 类似于java中的instence的恶汉式

	class App : Application() {
		val database: SQLiteOpenHelper by lazy {
			MyDatabaseHelper(applicationContext)
		}

		override fun onCreate() {
			super.onCreate()
			val db = database.writableDatabase
		}
	}

database并没有被真正初始化，直到第一次调用 onCreate  时才会执行 MyDatabaseHelper(applicationContext) 来进行初始化,  lazy  操作符是线程安全的。

>如果不担心多线程问题或者想提高更多的性能，也可以使用 lazy(LazyThreadSafeMode.NONE){ ... } 。

>lazy 的函数内部表示的是 初始化 ,例如

	val a = 5  

>这样表示 直接给a初始化为5 ,不论a是否用到

	val a: Int by lazy {
        5
    }

>这样表示 a 是没有初始化的, 只有当使用a的时候 才会对a进行赋值,此时的a就被赋值了

#### Observable ####

这个委托会帮我们监测我们希望观察的属性的变化。当被观察属性的 set  方法被调用的时候，它就会自动执行我们指定的lambda表达式。所以一旦该属性被赋了新的值，我们就会接收到被委托的属性、旧值和新值。

	class ViewModel(val db: MyDatabase) {
		var myProperty by Delegates.observable("") {
			d, old, new ->
			db.saveChanges(this, new)
		}
	}

>这个例子展示了，一些我们需要关心的ViewMode，每次值被修改了，就会保存它们到数据库。

		var my: String by Delegates.observable("") { d, old, new ->
            println("d=$d----old=$old----new=$new")
        }
        println("my=$my")
        my = "是镂空的解放路口时代峻峰"
        println("my=$my")

>参数有两个 , 1, my的初始化值,等于 oid的值 , 2,lamadas表达式 . 
>结果的参数有三个 1,d表示代理的名字, 结果为 my .2,oid表示 之前的值. 3,new表示变化后的值

#### Vetoable ####

这是一个特殊的 observable  ，它让你决定是否这个值需要被保存。它可以被用于在真正保存之前进行一些条件判断。

	var positiveNumber = Delegates.vetoable(0) {
		d, old, new ->
		new >= 0
	}
>上面这个委托只允许在新的值是正数的时候执行保存。在lambda中，最后一行表示返回值。但是不需要使用return关键字

#### Not Null ####

有时候我们需要在某些地方初始化这个属性，但是我们 1, 不能在构造函数中确定，或者 2 ,我们不能在构造函数中做任何事情。

第二种情况在Android中很常见：在Activity、fragment、service、receivers……无论如何，一个非抽象的属性在构造函数执行完之前需要被赋值。为了给这些属性赋值，我们无法让它一直等待到我们希望给它赋值的时候。我们至少有两种选择方案。

第一种就是使用可null类型并且赋值为null，直到我们有了真正想赋的值。但是我们就需要在每个地方不管是否是null都要去检查。如果我们确定这个属性在任何我们使用的时候都不会是null，这可能会使得我们要编写一些必要的代码了。

第二种选择是使用 notNull  委托。它会含有一个可null的变量并会在我们设置这个属性的时候分配一个真实的值。如果这个值在被获取之前没有被分配，它就会抛出一个异常。


	class App : Application() {
		companion object {
			var instance: App by Delegates.notNull()
		}

		override fun onCreate() {
			super.onCreate()
			instance = this
		}
	}

### 从Map中映射值 ###

另外一种属性委托方式就是，属性的值会从一个map中获取value，属性的名字对应这个map中的key。这个委托可以让我们做一些很强大的事情，因为我们可以很简单地从一个动态地map中创建一个对象实例。

	import kotlin.properties.getValue

	class Configuration(map: Map<String, Any?>) {
		val width: Int by map
		val height: Int by map
		val dp: Int by map
		val deviceName: String by map
	}

>对于这个类去创建一个必须要map 的方法：

	conf = Configuration(mapOf(
		"width" to 1080,
		"height" to 720,
		"dp" to 240,
		"deviceName" to "mydevice"
	))

###创建一个自定义的委托##

创建一个 Single  的委托，它只能被赋值一次，如果第二次赋值，它就会抛异常。

Kotlin库提供了几个接口，我们自己的委托必须要实现： ReadOnlyProperty(只读)  和 ReadWriteProperty(可读可写)  。具体取决于我们被委托的对象是 val  还是 var  。

1, 创建一个类然后继承 ReadWriteProperty  ：

	private class NotNullSingleValue<T>() : ReadWriteProperty<Any?, T> {
		private var value: T? = null
	    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
	        return value ?: throw IllegalStateException("not initialized")
	    }
	
	    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
	       	this.value = if (this.value == null) value
        	else throw IllegalStateException("already initialized")
	    }
	}	

这个委托可以作用在任何非null的类型。它接收任何类型的引用，然后像getter和setter那样使用T。现在我们需要去实现这些函数。

- Getter函数 如果已经被初始化，则会返回一个值，否则会抛异常。
- Setter函数 如果是null，则赋值，否则会抛异常。

创建一个对象，然后添加函数使用委托：

	object DelegatesExt {
		fun notNullSingleValue<T>():
			ReadWriteProperty<Any?, T> = NotNullSingleValueVar()
	}



### 创建一个SQLiteOpenHelper ###

#### ManagedSqliteOpenHelper ####

Anko提供了很多强大的SqliteOpenHelper来可以大量简化代码。
使用 ManagedSqliteOpenHelper  我们只需要：

	forecastDbHelper.use {
		...
	}


### 依赖注入 ###



## 集合和函数操作符 ##

关于函数式编程很不错的一点是我们不用去解释我们怎么去做，而是直接说我想做什么。

比如，如果我想去过滤一个list，不用去创建一个list，遍历这个list的每一项，然后如果满足一定的条件则放到一个新的集合中，而是直接使用 filer 函数并指明我想用的过滤器。用这种方式，我们可以节省大量的代码。

kotliin中关于集合操作的 本地接口 有:

- **Iterable**:父类.所有我们可以遍历的一系列工具都是实现这个接口。
- **MutableIterable**:一个支持遍历的同时可以执行增加和删除的Iterables。 MutableIterable<out T> : Iterable<T>
- **Collection**:这个类相是一个泛型集合。我们通过函数访问可以返回集合的size、是否为空、是否包含一个或者一些item。这个集合的所有方法提供查询，因为connections是不可修改的。
- **MutableCollection**:一个支持增加和删除item的Collection。它提供了额外的函数，比如 add  、 remove  、 clear  等等。
- **List**:可能是最流行的集合类型。它是一个泛型有序的集合。因为它的有序，我们可以使用 get  函数通过position来访问。
- **MutableList**:一个支持增加和删除item的List。
- **Set**:一个无序并不支持重复item的集合。
- **MutableSet**:一个支持增加和删除item的Set集合
- **Map**:一个key-value对的collection。key在map中是唯一的，也就是说不能有两对key是一样的键值对存在于一个map中。
- **MutableMap**:一个支持增加和删除item的map.

有很多不同集合可用的函数操作符。

### 总数操作符 -- 一般对 list中item中为 Int 的集合进行操作 ###

#### **any** ####

如果至少有一个元素复合给出的判断条件,就返回true

	val list = listof(1,2,3,4,5,6)
	val a = list.any { it % 2 == 0 }
	val b = list.any { it > 10 }

>any 方法表示任何一个, 其参数接收一个 函数,改函数是一个条件, 这个方法的意味着便利集合,如果 集合中 有一个item 的元素符合该函数条件,就返回true,否则返回false. 每个item 可以使用it表示 ,kotlin 中的条目 it 是 一个函数内只有一个参数时对这个参数的简称
>>any还有一个重载函数没有任何参数 , 如果是集合则返回 是否不为空,如果不是是集合则返回 是否有值

#### **all** ####
如果全部的元素符合给出的判断条件，则返回true。

	val a = list.all { it % 2 == 0 }
	val b = list.all { it < 10 }

>all 方法表示便利所有的 item ,如果全部满足 条件返回true ,否则返回false

#### **count** ####
返回符合给出判断条件的元素总数。

	val a = list.count { it % 2 == 0 }
>count方法表示 遍历item 后复合条件的个数
>>count方法有个重载函数是没有任何参数的,返回的是集合的size

#### **fold** ####
在一个初始值的基础上从第一项到最后一项通过一个函数累计所有的元素。

	val a = list.fold(1) { acc, i -> acc + i }

#### **foldRight** ####
与 fold  一样，但是顺序是从最后一项到第一项。

#### **forEach** ####
遍历所有元素，并执行给定的操作。

	list.forEach { println(it) }

#### **forEachIndexed** ####
与 forEach  ，但是我们同时可以得到元素的index。

	list.forEachIndexed { index, i -> println("第${index}个值的结果为$i") }

#### **max** ####
返回最大的一项，如果没有则返回null。

#### **maxBy** ####
根据给定的函数返回最大的一项，如果没有则返回null。

	list.maxBy { -it }

>maxBy的过程是 将每一个item 通过函数中的方法进行转换后,再进行比较, 但是如果函数中没有对 it 或者 i-> 进行转换 ,那就相当于没有对数值进行转换

#### **min** ####	
返回最小的一项，如果没有则返回null。

#### **minBy** ####
根据给定的函数返回最小的一项，如果没有则返回null。

#### **none** ####
如果没有任何元素与给定的函数匹配，则返回true。

	list.none()
	list.none { it % 3 == 0 }

>none代表如果没有符合条件就返回true 
>>其重载函数返回 如果为空 就返回true ,如果不是集合 就判断如果没有值就返回true

#### **reduce** ####

与 fold  一样，但是没有一个初始值。通过一个函数从第一项到最后一项进行累计。

	val a = list.reduce{ acc, i -> acc + i }

#### **reduceRight** ####
与 reduce  一样，但是顺序是从最后一项到第一项。

#### **sumBy** ####
返回所有每一项通过函数转换之后的数据的总和。

	list.sumBy { it / 2 }

### 过滤操作符 ###

**分为 drop (去除---一般用于去除某个或某些不符合条件的,根据item处理) 和 filter(过滤 --- 一般用于获取某个或某些符合条件的,根据item处理) 和 take(获取 --- 一般用于根据角标获取数据,根据角标进行处理)三大类	**

#### **drop(去除)** ####
返回包含去掉前n个元素的所有元素的列表。

	list.drop(3).forEach { println("number=$it") }
>返回去掉前三个item 后的集合 即 第4项到最后一项的集合

#### **dropWhile** ####
返回根据给定函数条件从第一项开始去掉符合条件元素的列表。

	list.dropWhile { it < 4 }.forEach { println("number=$it") }

#### **dropLastWhile , dropLast** ####
返回根据给定函数从最后一项开始去掉指定元素的列表。

#### **filter(筛选)** ####
过滤出所有符合给定函数条件的元素。(过滤出符合的取出来)

	list.filter { it > 4 }.forEach { println("number=$it") }
>返回的参数为 item 的具体值,这个值和 position 无关, filter 方法的过滤条件表示 一个 list中符合条件的, 是将 符合条件的取出来而非过滤掉 ,也就是说拿到的结果是所有符合条件的元素集合

#### **filterNot** ####
过滤掉出所有符合给定函数条件的元素。 (过滤掉符合的,剩下不符合条件的)

>与 filter相反, 这个方法会把 符合条件的过滤掉, 把剩下的取出来,与 filter 互补

#### **filterNotNull** ####
过滤掉所有元素中不是null的元素。

>这个方法是 filterNot 的进阶版, 只是条件是 it==null ,即 过滤掉符合 item为空的,剩下 item != null 的

#### **slice** ####
过滤出一个list中指定index的元素。

	list.slice(listOf(2, 4, 5)).forEach { println("number=$it") }
>slice接收一个 数组 或 集合 参数, 改集合的值代表 list的角标.返回list中这个角标集合的值 . 实际使用的时候 不会处理角标越界的问题 ,所以是有可能出现角标越界的

#### **take** ####
返回从第一个开始的n个元素。即前 n 个元素

	list.take(5).forEach { println("number=$it") }

>这个take 主要是通过角标获取值, 相当于 slice 方法的参数为 0--n

#### **takeLast** ####
返回从最后一个开始的n个元素

#### **takeWhile , takeLastWhile** ####
返回从第一个开始(takeWhile)或者从最后一个开始(takeLastWhile) 直到首个不符合条件的 的元素集合。

	list.takeWhile { it <3 }.forEach { println("number=$it") }

>这个传过来的参数是 item 而非角标 , 当item符合的时候才会会继续判断下一个, 直至首个不符合条件的会 break 返回, 即如果第一个 item 不符合 条件,即使后面的所有item都符合,也会返回一个空集合

### 映射操作符 ###

#### **flatMap** ####
遍历所有的元素，为每一个创建一个集合，最后把所有的集合放在一个集合中。

	list.flatMap { listOf(it, it + 1) }.forEach { println("number=$it") }
>结果为: listOf(1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7)

>调用flatmap的时候 会遍历 list中的数据 ,然后每个 item 值会在 listOf(it , it+1) 中进行转换形成新的 list, 即 会形成 list.size 个分集合, 每个集合 都是 item 形成的子集合, 最终把所有的集合合并, 返回这个最终的集合.

#### **groupBy** ####
返回一个根据给定函数分组后的map。条件会返回 key ,所有符合这个key的value会形成一个集合

	list.groupBy { if(it%2==0)"event" else "oil" }.forEach { println("number=${it.key}--${it.value}") }

>groupBy会遍历每个item , 然后通过 it 来指代这个item进行区分, 如果不指定默认使用 Koilin.Unit 对item区分后会分成不同组
>>使用groupBy 最终会生成 key: Any 和 value:List<E> 的map对象

#### **map -- 转换** ####
返回一个每一个元素根据给定的函数转换所组成的List。

	list.map { it * 2 }.forEach { println("number=$it") }

>groupBy返回map集合, 但是 map方法会返回list集合 , map 的含义是 , 遍历list中的每个item ,然后将 item 根据给定的函数进行转换, 即 最终行程的集合是  list中的每个item 根据函数转换后 形成的集合.

#### **mapIndexed** ####
返回一个每一个元素根据给定的包含元素index的函数转换所组成的List。

	list.mapIndexed { index, i -> index + i }.forEach { println("number=$it") }

	list.mapIndexed { _, i -> i *2}.forEach { println("number=$it") }

>mapindexed 方法会吧 index 也传入参数中, 相对于 map多了一个 index 角标的可用参数, 如果 index 参数不用的话 可以简写成 _ 即 {_,i->index*2} , 一般的如果 index参数不用的话 可以使用 map方法而非 mapIndexed方法

#### **mapNotNull** ####
返回一个每一个非null元素根据给定的函数转换所组成的List。

	list.mapNotNull { it + 1 }.forEach { println("number=$it") }

>mapNotNull方法最终会形成一个集合 , 该集合首先会排除 null元素, 然后将非null元素根据函数进行转换 . 如果没有任何函数进行转换, 可以理解为 只是将list中所有item 为null 的条目进行删除.当然 如果 不需要任何 转换的话 可以使用 filterNotNull 方法

### 元素操作符 -- 主要是对 list 中对 item 进行查找操作 ###

#### **contains -- 包含** ####
如果指定元素可以在集合中找到，则返回true。底层调用 了 list.indexOf(object o)

#### **elementAt -- 通过 positon 获取** ####
返回给定index对应的元素，如果index数组越界则会抛出 IndexOutOfBoundsException  。底层中会调用 get方法,所以一般不使用这个方法
>一般不用这个方法, 一般都使用 list.get(pos) 或者 list[pos]

#### **elementAtOrElse** ####
返回给定index对应的元素，如果index数组越界则会根据给定函数返回默认值。

	println("${list.elementAtOrElse(4) { 0 }}")

>如果能取到 角标为 4 的值则返回, 如果数组越界, 则 返回默认的值, 这个值不是角标的值, 是直接返回该值 , 即如果数组越界结果就是这个默认的值 .因为如果参数是 角标的化 那么 这个默认的角标仍然是有可能越界的

#### **elementAtOrNull** ####
返回给定index对应的元素，如果index数组越界则会返回null。

#### **first** ####
返回符合给定函数条件的第一个元素。 函数 会将item作为参数对item进行筛选

	println("${list.first { it > 3 }}")

#### **firstOrNull** ####
返回符合给定函数条件的第一个元素，如果没有符合则返回null。

#### **indexOf** ####
返回指定元素的第一个index 角标，如果不存在，则返回 -1  。
	println("${list.indexOf(5)}")
>就是 list.indexOf(object o) 获取某个对象的 角标.

#### **indexOfFirst** ####
返回第一个符合给定函数条件的元素的index，如果没有符合则返回 -1  。

>返回 第一个 对 item进行函数转换后的角标

#### **last** ####
返回符合给定函数条件的最后一个元素。 

	println("${list.last { it % 2 == 0 }}")

>元素和角标需要分清楚, 因为一般返回的都是 元素, 即 item 而非 角标 position.

#### **lastIndexOf** ####

返回指定元素的最后一个index，如果不存在，则返回 -1  。返回 object 的index

#### **lastOrNull** ####
返回符合给定函数条件的最后一个元素，如果没有符合则返回null。

#### **single** ####
返回符合给定函数的单个元素，如果没有符合或者超过一个，则抛出异常。
	list.single { it % 5 == 0 }
>这个一般用不到

#### **singleOrNull** ####
返回符合给定函数的单个元素，如果没有符合或者超过一个，则返回null。

### 生产操作符 ###

#### **merge** ####
把两个集合合并成一个新的，相同index的元素通过给定的函数进行合并成新的元素作为新的集合的一个元素，返回这个新的集合。新的集合的大小由最小的那个集合大小决定。

	assertEquals(listOf(3, 4, 6, 8, 10, 11), list.merge(listRepeated) { it1, it2 -> it1 + it2 })
	
>使用 merge 来讲两个集合合并成一个集合, 该集合size 是 最小的那个集合的size
>新集合的每个元素 是通过 相同index 的相对元素来进行处理的.

#### **partition** ####
把一个给定的集合分割成两个，第一个集合是由原集合每一项元素匹配给定函数条件返回 true  的元素组成，第二个集合是由原集合每一项元素匹配给定函数条件返回 false  的元素组成。

	assertEquals(
		Pair(listOf(2, 4, 6), listOf(1, 3, 5)), list.partition { it % 2 == 0 }
	)

#### **plus** ####
返回一个包含原集合和给定集合中所有元素的集合，因为函数的名字原因，我们可以使用 +  操作符。

	assertEquals(
		listOf(1, 2, 3, 4, 5, 6, 7, 8),list + listOf(7, 8)
	)
>因为 plus 方法是用 operter 修饰的,可以使用运算操作符替代

#### **zip** ####
返回由 pair  组成的List，每个 pair  由两个集合中相同index的元素组成。这个返回的List的大小由最小的那个集合决定。

	assertEquals(
		listOf(Pair(1, 7), Pair(2, 8)), list.zip(listOf(7, 8))
	)

#### **unzip** ####
从包含pair的List中生成包含List的Pair。

	assertEquals(
		Pair(listOf(5, 6), listOf(7, 8)),listOf(Pair(5, 7), Pair(6, 8)).unzip()
	)


### 顺序操作符 ###

#### **reverse -- 反转** ####
返回一个与指定list相反顺序的list。

	val unsortedList = listOf(3, 2, 7, 5)
	assertEquals(listOf(5, 7, 2, 3), unsortedList.reverse())

#### **sort -- 排序** ####
返回一个自然排序后的list。

	assertEquals(listOf(2, 3, 5, 7), unsortedList.sort())

#### **sortBy** ####
返回一个根据指定函数排序后的list。

	assertEquals(listOf(3, 7, 2, 5), unsortedList.sortBy { it % 3 })

>这个是通过item进行计算后再根据结果进行排序,最终返回其真实元素

#### **sortDescending** ####
返回一个降序排序后的List。

	assertEquals(listOf(7, 5, 3, 2), unsortedList.sortDescending())

#### **sortDescendingBy** ####
返回一个根据指定函数降序排序后的list。

	assertEquals(listOf(2, 5, 7, 3), unsortedList.sortDescendingBy {it % 3 })









