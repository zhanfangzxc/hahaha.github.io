title: 'Retrofit2.0:Android最好用的http库有史以来最大的更新'
date: 2015-09-16 13:42:02
tags:
- Android
- Retrofit 2.0
- Http

---

**翻译原文**:[http://inthecheesefactory.com/blog/retrofit-2.0/en](http://inthecheesefactory.com/blog/retrofit-2.0/en)
> 介绍

Retrofit由于其相比于其他http开源库具有更加简单和出色的性能，从而成为Android端最流行的Http客户端库之一。

存在的缺点是在1.X版本上没有取消正在进行中的事务的方法，如果你想做这个操作必须手动在其他线程中杀死他，所以很难实现。

Square公司多年之前就承诺Retrofit 2.0很快就可以使用了，但是多年过去了，仍然没有任何更新的消息。

直到上周，Retrofit 2.0从候补阶段变成了Beta 1，而且向每个人公布了，自从尝试了之后，我不得不说Retrofit 2.0有很多令我印象深刻的地方，有很多方法的变化，我将在这篇文章中描述那些改变的东西，让我们开始吧。

> 还是之前的包名，只是换了新的版本号

如果你想在你的项目中引入Retrofit 2.0，添加下面这行代码在你的**build.gradle**文件中的**dependencies**节点

```
compile 'com.squareup.retrofit:retrofit:2.0.0-beta1'
```
同步你的gradle文件然后你就可以使用Retrofit 2.0

> 新的服务声明方式，没有同步和异步之分

关于在Retrofit 1.9中生命服务借口，如果你想声明一个同步的方法，你要像下面这样声明：

```
/* Synchronous in Retrofit 1.9 */
 
public interface APIService {
 
    @POST("/list")
    Repo loadRepo();
 
}
```

如果你想声明一个异步的就需要像下面这样

```
/* Asynchronous in Retrofit 1.9 */
 
public interface APIService {
 
    @POST("/list")
    void loadRepo(Callback<Repo> cb);
 
}
```
但是如果在Retrofit 2.0上，声明方式只有一种，所以更加简单了

```
import retrofit.Call
/*Retrofit 2.0*/

public interfase APIService{
	@POST("/list")
	Call<Repo> loadRepo();
}
```
调用创建的方法和OKHttp的调用模式是一样的，如果异步的请求，只要调用**excute**或者**enqueue**来创建一个异步的请求。

异步请求的调用

```
Call<Repo> call = service.loadRepo();
Repo repo = call.excute();
```
上面的代码会锁定当前县城，所以在Android中不能在主线程中调用，否则你会面临**NetworkOnMainThreadException**，如果你想执行**excute**方法，你必须在其他的线程中执行。

**异步请求**

```
Call<Repo> call = service.loadRepo();
call.enqueue(new Callback<Repo>(){
	@Override
	public void onResponse(Response<Repo> response){
		//从response.body()中获取结果
	}
	@Override
	public void onFailure(Throwable t){
	
	}
});
```

上面的代码将会在后台线程创建ygie请求用来从响应的**response.body()**方法中获取请求结果。但是请注意**onResponse**和**onFailure**方法将在主线程中调用。

所以我建议你使用**enqueue**，他很好的适用Android的行为。

> 取消正在进行中的请求

服务模式变成Call的原因是正在进行中的请求事务可以被取消，只用简单的使用**call.cancel()**就可以了。

```
call.cancel();
```
调用这个方法之后，事务立马就可以被取消了，是不是很简单。

> 新的服务的创建，转换器已经被Retrofit排除在外了

在Retrofit 1.9中，GsonConverter被包含在了Retrofit包中，当RestAdapter一创建的时候就可以自动初始化，所以，如果是json的结果可以自动被转换成我们定义的数据访问对象。

但是在Retrofit 2.0中，转换器将不在包含在Retrofit包中，所以需要手动的插入一个转换器在Retrofit中，否则只能接受String结果，所以， Retrofit 2.0不再依赖Gson。

如果你想将接收到的结果转换成数据访问对象，你必须添加Gson引用和创建一个Gson转换器

```
compile 'com.squareup.retrofit:converter-gson:2.0.0-beta1'
```

然后通过**addConverterFactory**添加Gson转换器，值得注意的是之前的**RestAdapter**已经改名为**Retrofit**。

```
Retrofit retrofit = new Retrofit.Builder()
			.baseUrl("http://api.nuuneoi.com/base/")
			.addConverterFactory(GsonConverterFactory.create())
			.build();
service = retrofit.create(APIService.class);
```
以下是由Square公司提供的转换器，选择一个适合你的：

```
Gson: com.squareup.retrofit:converter-gson
Jackson: com.squareup.retrofit:converter-jackson
Moshi: com.squareup.retrofit:converter-moshi
Protobuf: com.squareup.retrofit:converter-protobuf
Wire: com.squareup.retrofit:converter-wire
Simple XML: com.squareup.retrofit:converter-simplexml
```
你也可以通过实现**Converter.Factory**接口来创建自己的转换器。

我比较赞成这种新的模式，因为让Retrofit对于要做的事情看起来更加清晰。

> 自定义Gson对象

如果你需要在json中进行一些格式化，比如，日期格式化，你可以通过**GsonconverterFactory.create()**创建一个Gson对象：

```
Gson gson = new GsonBuiler()
		.setDateFormat("yyyy-MM-dd'T'HH:mm:ssZ")
		.create();
Retrofit retrofit = new Retrofit.Bilder()
				.baseUrl("http://api.nuuneoi.com/base/")
				.addConverterFactory(GsonConverterFactory.create(gson))
				.build();
service = retrofit.create(APIService.class);
```
> 新的Url解决方式，就像<a href></a>那样

Retrofit 2.0提出了一个心的URL解决概念，不再是之前简单的将BaseUrl和@Url合并在一起，而是像<a href="..."></a>那样，请先看一下下面的示例:

![示例图片](http://inthecheesefactory.com/uploads/source/blog/retrofit2/apiservice1.png)
![示例图片](http://inthecheesefactory.com/uploads/source/blog/retrofit2/apiservice2.png)
![示例图片](http://inthecheesefactory.com/uploads/source/blog/retrofit2/apiservice3.png)

下面是我的一些对于Retrofit 2.0 URL声明模式的建议：

```
Base URL：永远用**/**结尾

@URL 不要用**/**开头
```
示例：

```
public interface APIService{
	@POST("user/list")
	Call<Users> loadUsers();
}

public void doSomething(){
	Retrofit retrofit = new Retrofit.Builder()
			.baseUrl("http://api.nuuneoi.com/base/")
			.addConverterFactory(GsonConverterFactory.create())
			.build();
			
	APIService service = retrofit.create(APIService.class);
}
```
上面的代码将从http://api.nuuneoi.com/base/user/list获取数据

我们也可以在Retrofit 2.0中声明完整的URL

```
public interface APIService{
	@POST("http://api.nuuneoi.com/special/user/list")
	Call<Users> loadSpecialUsers();
}
```
在这个示例中BaseUrl将会被忽略。

和之前的版本相比，你将会看到一个主要的改变，如果你想将自己的项目中的改成Retrofit 2.0，不要忘记修改这些URL部分的代码。

> OkHttp现在是必须的

在Retrofit 1.9上，OkHttp是可选的，如果你想使用OKHttp作为网络连接接口的话，你必须手动的把OKHttp的引用添加进你的项目。

但是在Retrofit 2.0上，OKHttp是必须的，而且会自动引入你的项目中。在Retrofit 2.0上，OkHttp是会自动作为Http连接接口使用的。

> onResponse方法仍然会被调用尽管Response存在一些问题

在Retrofit 1.9上，如果你获取到的响应不能被转换成所定义的对象的话。将会调用failure方法，但是在Retrofit 2.0，不管获取到的响应信息能否转换成定义的类型，都会调用onResponse方法，但是当响应信息不能转换的时候，**response.body()**将会返回null，所以不要忘记处理。

如果响应中存在任何的问题，比如：404，**onResponse**都会被调用，所以你可以通过**response.errorBody().string()**中获取错误信息。

![错误信息示例](http://inthecheesefactory.com/uploads/source/blog/retrofit2/response.jpg)

Response/Failure的逻辑跟Retrofit 1.9有很大的不同，所以如果你决定切换到Retrofit 2.0的时候一定要小心处理这些逻辑。

> 没有网络连接权限引起的SecurityException

在Retrofit 1.9上，如果你忘记在**AndroidManifest.xml**中添加网络权限的话，异步的请求将会直接调用**failure**方法，并且会提示没有权限的错误信息，不会有其他的异常抛出。

但是在Retrofit 2.0上，当你调用**call.enqueue**或者**call.execute**,然后就会直接抛出**SecurityException**异常，如果你不用try-catch解决的话，程序会直接终止。

![错误信息提示](http://inthecheesefactory.com/uploads/source/blog/retrofit2/sec.png)

这个行为就如同你手动的调用**HttpURLConnection**,这只是个小问题，只要你将权限添加进**AndroidMKanifest.xml**文件中就行了。

> 使用OKHttp的拦截器

在Retrofit 1.9上你可以使用请求拦截器去拦截一个请求，但是自从Http连接层移到OKHttp上之后，在2.0中拦截器已经被移除掉了。

所以我们现在必须使用OKHttp的拦截器，首先你需要创建一个带有拦截器的OKHttpClient对象，如下：

```
OkHttpClient client = new OkHttpClient();
client.interceptors.add(new Interceptor(){
	@Override
	public Response intercept(Chain chain) throws IOException{
		return response;
	}
});
```
然后将上面创建的**client**添加到Retrofit的生成器链中。

```
 Retrofit retrofit = new Retrofit.Builder()
 			.baseUrl("http://api.nuuneoi.com/base")
 			.addConverterFactory(GsonConverterFactory.create())
 			.client(client)
 			.build();
```
如果想了解更多关于OKHttp拦截器的话，请看[OKHttp interceptors](https://github.com/square/okhttp/wiki/Interceptors)

> 在CallAdapter中集成RxJava

相比声明一个**Call<T>**接口，。我们更愿意以我们自己的方式声明，比如：**MyCall<T>**,这就是Retrofit 2.0的**CallAdapter**机制。

Retrofit团队提供了一些可用的CallAdapter模块，最流行的一个莫过于集成RxJava的CallAdapter，返回类型是**Observable<T>**,如果想使用这个，下面的两个模块必须引入项目中：

```
compile 'com.squareup.retrofit:adapter-rxjava:2.0.0-beta1'
compile 'io.reactivex:rxandroid:1.0.1'
```

同步一下Gradle，然后在Retrofit创建链中调用**addCallAdapterFactory**方法

```
Retrofit retrofit = new Retrofit.Builder()
		 .baseUrl("http://api.nuuneoi.com/base/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
        .build();
```

现在服务接口将会返回**Observable<T>**:

```
public interface APIService{
	@POST("list")
	Call<DessertItemCollectionDao> loadDessertList();
	
	@POST("list")
	Observable<DessertItemCollectionDao> loadDessertListRx();
}
```
你可以以RxJava的方式使用，如果你想让subscribe部分的代码运行在主线程中，需要添加**observeOn(AndroidSchedulers.mainThread())**.

```
Observable<DessertItemCollectionDao> observable = service.loadDessertListRx();

observable.observeOn(AndroidSchedulers.mainThread())
	.subsribe(new Subscriber<DessertItemCollectionDao>(){
	@Override
	public void onCompleted(){
		 Toast.makeText(getApplicationContext(),
                    "Completed",
                    Toast.LENGTH_SHORT)
                .show();
        }
 
        @Override
        public void onError(Throwable e) {
            Toast.makeText(getApplicationContext(),
                    e.getMessage(),
                    Toast.LENGTH_SHORT)
                .show();
        }
 
        @Override
        public void onNext(DessertItemCollectionDao dessertItemCollectionDao) {
            Toast.makeText(getApplicationContext(),
                    dessertItemCollectionDao.getData().get(0).getName(),
                    Toast.LENGTH_SHORT)
                .show();
        }
    });
```

> 结论

还有一些其他的改变，你可以去看官方的[更新文档](https://github.com/square/retrofit/blob/master/CHANGELOG.md)，我相信我已经把主要的更改在这篇文章中描述清楚了。

你也许会好奇什么时候可以把项目替换成Retrofit 2.0，因为他还处于测试阶段，而且你也可能像我一样是一个试用者，Retrofit 2.0没有过bug的工作良好也只是基于我的试验。

请注意Retrofit 1.9的官方文档已经从Square的github网站上移除了，我建议就从现在开始学习Retrofit 2.0，并且考虑在不就的将来替换成这个新的版本。