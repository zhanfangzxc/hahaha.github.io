title: Retrofit2 更新指南
date: 2015-10-12 09:39:49
categories: Android
tags:
- Android
- Retrofit

---
原文链接:[https://futurestud.io/blog/retrofit-2-upgrade-guide-from-1-9/](https://futurestud.io/blog/retrofit-2-upgrade-guide-from-1-9/)

### Retrofit概述

1. [Getting Started and Create an Android Client](https://futurestud.io/blog/retrofit-getting-started-and-android-client/)
2. [Basic Authentication with Retrofit](https://futurestud.io/blog/android-basic-authentication-with-retrofit/)
3. [OAuth with Retrofit](https://futurestud.io/blog/oauth-2-on-android-with-retrofit/)
4. [Token Authentication with Retrofit](https://futurestud.io/blog/retrofit-token-authentication-on-android/)
5. [Synchronous and Aynchronous Request Execution with Retrofit](https://futurestud.io/blog/retrofit-synchronous-and-asynchronous-requests/)
6. [Multiple Query Parameters of Same Name](https://futurestud.io/blog/retrofit-multiple-query-parameters-of-same-name/)
7. [Send Objects in Request Body with Retrofit](https://futurestud.io/blog/retrofit-send-objects-in-request-body/)
8. [Define Custom JSON Converter](https://futurestud.io/blog/retrofit-replace-the-integrated-json-converter/)
9. [Add Custom HTTP Request Headers](https://futurestud.io/blog/retrofit-add-custom-request-header/)
10. [Optional Query Parameters](https://futurestud.io/blog/retrofit-add-custom-request-header/)
11. [XML Converter with Retrofit](https://futurestud.io/blog/retrofit-how-to-integrate-xml-converter/)
12. [Debugging Requests with Retrofit's Log Level](https://futurestud.io/blog/retrofit-using-the-log-level-to-debug-requests/)
13. [File Upload with Retrofit](https://futurestud.io/blog/retrofit-how-to-upload-files/)
14. [Round-Up Retrofit Series](https://futurestud.io/blog/retrofit-series-round-up/)
15. [Retrofit Upgrade Guide: 1.9 to 2.0](https://futurestud.io/blog/retrofit-2-upgrade-guide-from-1-9/)

### 添加Gradle依赖项

	compile 'com.squareup.retrofit:retrofit:2.0.0-beta2'
	compile 'com.square.okhttp:okhttp:2.5.0'
	
Retrofit 2默认情况下不集成Gson，在之前，你不用去考虑转换器的集成问题，现在的变化需要你导入你需要的转换器开源库，接下来将会给你展示如何配置Gson或其他的转换器。

### 转换器

	comile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'
	
默认情况戏Rxjava也不会引入，所以你需要手动的添加Rxjava的依赖项

	compile 'com.squareup.retrofit:adapter-rxjava:2.0.0-beta2'
	compile 'io.reativex:rxandroid:1.0.1'
	
### RestAdapter -> Retrofit

之前的**RestAdapter**重新命名为**Retrofit**

**Retrofit 1.9**

	RestAdapter.Builder builder = new RestAdapter.Builder();

**Retrofit 2.0**

	Retrofit.Builder builder = new Retrofit.Builder();
	
### setEndPoint -> baseUrl

**setEndPoint**方法变成了**baseUrl**

**Retrofit 1.9**

	RestAdapter.Builder builder = new RestAdapter.Builder()
		.setEndpoint(API_BASE_URL)
		.build();
		
	YourService service = adapter.create(YourService.class);
	

**Retrofit 2.0**

	Retrofit.Builder builder = new Retrofit.Builder()
		.baseUrl(API_BASE_URL)
		.build();
		
	YourService service = retrofit.create(YourService.class);
	
### Base Url 处理

在之前的版本，定义的endpoint会作为默认的请求url，你自己定义的那部分包括查询或路径参数，请求体或multiparts，所以最后的请求url是endpoint url和自己定义的那部分url的结合：

**Retrofit 1.x**

	public interface UserService{
		@POST("me")
		User me();
	}
	
	RestAdapter adapter = RestAdapter.Builder()
		.baseUrl("http://your.api.url/v2/");
		.build();
		
	UserService service = adapter.create(UserService.class);
	
	//所以对于service.me()方法的请求url就是：
	//Https://your.api.url/v2/me
	
**Retrofit 2.x**

	public interface UserService(){
		@POST("/me")
		User me();
	}
	
	Retrofit retrofit = Retrofit.Builder()
		.baseUrl("https://your.api.url/v2");
		.build();
		
	UserService service = retrofit.create(UserService.class);
	
	// 最终service.me()方法的请求url是：
	// http://your.api.url/me
	
第二种样子

	public interface UserService {  
	    @POST("me")
	    User me();
	}

	Retrofit retrofit = Retrofit.Builder()  
	    .baseUrl("https://your.api.url/v2/");
	    .build();

	UserService service = retrofit.create(UserService.cass);

	// the request url for service.me() is: 
	// https://your.api.url/v2/me
	
### 动态Url

	public interface UserService{
		@GET
		public Call<File> getZipFile(@Url String url);
	}
	
### OkHttp请求

	compile 'com.squareup.okhttp:okhttp:2.5.0'
	
### OkHttp拦截器

	OkHttpClient client = new OkHttpClient();  
	client.interceptors().add(new Interceptor() {  
	    @Override
	    public Response intercept(Chain chain) throws IOException {
	        Request original = chain.request();

	        // Customize the request 
	        Request request = original.newBuilder()
	                .header("Accept", "application/json")
	                .header("Authorization", "auth-token")
	                .method(original.method(), original.body())
	                .build();

	        Response response = chain.proceed(request);

	        // Customize or return the response 
	        return response;
	    }
	});

	Retrofit retrofit = Retrofit.Builder()  
	    .baseUrl("https://your.api.url/v2/");
	    .client(client)
	    .build();

如果你要使用自定义的OkHttpClient，你必须要设置在**Retrofit.Builder**中设置Client通过**.client()**方法，这将会更新默认的Client。

### 同步和异步请求

同步方法需要一个返回类型，异步方法需要一个Callback作为最后一个参数

在Retrofit 2中，同步和异步将没有什么区别，请求现在包裹成一个通用的类使用所需的响应类型。下面的内容将为你讲述Retrofit 1和Retrofit 2在service声明和请求执行方面的区别。

### Interface 声明

同步和异步请求的区别，你定义任何实际类型或者**Void**作为返回类型，异步需要你定义一个通用的回调作为最后一个方法参数，，下面的代码展示了一个Retrofit 1的声明方法：

**Retrofit 1.9**

	public interface UserService {  
	    // Synchronous Request
	    @POST("/login")
	    User login();

	    // Asynchronous Request
	    @POST("/login")
	    void getUser(@Query String id, Callback<User> cb);
	}
	
**Retrofit 2.x**

public interface UserService {  
    @POST("/login")
    Call<User> login();
}

### 请求执行

**Retrofit 1.9**

	// synchronous
	User user = userService.login();

	// asynchronous
	userService.login(new Callback<User>() {  
	@Override
	    public void success(User user, Response response) {
	        // handle response
	    }

	    @Override
	    public void failure(RetrofitError error) {
	        // handle error
	    }
	});
	
Retrofit 1和Retrofit 2的请求完全不同，在2中所有的请求都会包裹在一个**Call**对象中，你可以通过**call.execute()**执行同步请求，通过**call.enqueue(new Callback<>())**来执行异步请求，下面是示例代码：

	// synchronous
	Call<User> call = userService.login();  
	User user = call.execute();

	// asynchronous
	Call<User> call = userService.login();  
	call.enqueue(new Callback<User>() {  
	    @Override
	    public void onResponse(Response<User> response, Retrofit retrofit) {
	        // response.isSuccess() is true if the response code is 2xx
	        if (response.isSuccess()) {
	            User user = response.body();
	        } else {
	            int statusCode = response.code();

	            // handle request errors yourself
	            ResponseBody errorBody = response.errorBody();
	        }
	    }

	    @Override
	    public void onFailure(Throwable t) {
	        // handle execution failures like no internet connectivity 
	    }
	}
	
在Retrofit 2中即使请求不成功也会调用**onResponse()**.**Response**类有一个**isSuccess()**方法用来检查是否处理成功(返回**2xx**的状态码)，你可以利用响应对象来做进一步的处理，如果状态码不是2xx，你需要自行处理错误，如果你希望在失败的情况下得到一个响应体，那么你就需要通过**ResponseBody**类中的**errorBody()**方法来转换对象。

> 同步请求在Android上可能会导致应用程序崩溃在Android 4.0 +。使用异步请求来避免阻塞会导致应用程序的主UI线程失败。

### 取消请求

在Retrofit 1中没有提供取消请求的方法，在Retrofit 2中如果http调度程序没有执行他的时候，你可以取消任何请求。

	Call<User> call = userService.login();
	User user = call.execute();

	call.cancel();
	
### 没有默认的转换器

在之前的1的版本中默认有Gson做为json的转换器，在2的版本中没有默认的转换器，你需要自己定义转换器，如果你想使用Gson，就必须要在gradle添加Gson的依赖项：

	compile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'  
	
**可用的转换器**

- GSON: com.squareup.retrofit:converter-gson:2.0.0-beta2
- Moshi: com.squareup.retrofit:converter-moshi:2.0.0-beta2
- Jackson: com.squareup.retrofit:converter-jackson:2.0.0-beta2
- SimpleXML: com.squareup.retrofit:converter-simplexml:2.0.0-beta2
- ProtoBuf: com.squareup.retrofit:converter-protobuf:2.0.0-beta2
- Wire: com.squareup.retrofit:converter-wire:2.0.0-beta2

### 给Retrofit添加转换器

	Retrofit retrofit = Retrofit.Builder()  
	    .baseUrl("https://your.api.url/v2/");
	    .addConverterFactory(ProtoConverterFactory.create())
	    .addConverterFactory(GsonConverterFactory.create())
	    .build();
	    
### Rxjava 集成

Retrofit 1中集成了三中请求执行机制：同步，异步和Rxjava；在Retrofit 2中默认的只有同步和异步的请求，但是Retrofit提供了一个添加其他请求执行机制进Retrofit中的方法，你可以添加更多的机制在你的app中，比如说Rxjava

**Gradle 依赖**

	compile 'com.squareup.retrofit:adapter-rxjava:2.0.0-beta2'  
	compile 'io.reactivex:rxandroid:1.0.1'  
	
接下来需要在创建service实例之前添加一个**CallAdapter**进**Retrofit**对象

	Retrofit retrofit = new Retrofit.Builder()  
	    .baseUrl(baseUrl);
	    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
	    .addConverterFactory(GsonConverterFactory.create())
	    .build();
	   
具体代码：
 
	public interface UserService {  
	    @POST("/me")
	    Observable<User> me();
	}

	// this code is part of your activity/fragment
	Observable<User> observable = userService.me();  
	observable.observeOn(AndroidSchedulers.mainThread()).subscribe(new Subscriber<User>() {  
	    @Override
	    public void onCompleted() {
	        // handle completed
	    }

	    @Override
	    public void onError(Throwable e) {
	        // handle error
	    }

	    @Override
	    public void onNext(User user) {
	        // handle response
	    }
	});
	
### 附加资源

- [Jake Wharton’s talk @ Droidcon NYC 2015 on Retrofit 2](https://www.youtube.com/watch?v=KIAoQbAu3eA)
- [Details on OkHttp interceptors](https://github.com/square/okhttp/wiki/Interceptors)
- [Retrofit Converters](https://github.com/square/retrofit/tree/master/retrofit-converters)
- [RxAndroid on GitHub](https://github.com/ReactiveX/RxAndroid)
- [Retrofit CallAdapters](https://github.com/square/retrofit/tree/master/retrofit-adapters)
- [Retrofit Change Log](https://github.com/square/retrofit/blob/master/CHANGELOG.md)
