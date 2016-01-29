title: RxAndroid介绍
date: 2016-01-25 09:29:36
tags:
- Android
- RxAndroid

---
原文地址:[https://medium.com/crunching-rxandroid/crunching-rxandroid-intro-c27eb6f009ea#.4ailarz9d](https://medium.com/crunching-rxandroid/crunching-rxandroid-intro-c27eb6f009ea#.4ailarz9d)

几个月之前，我阅读了一系列神奇的文章，一段时间之后，我决定自己尝试一下这个库，并且从中分享一下我每天所获得的知识。

### 现在让我们开始吧

首先，我们需要创建一个新的Android项目，添加RxAndroid的依赖到build脚本中

	compile 'io.reactivex:rxandroid:0.24.0'
	
如果你想跳过这些初始化的部分，可以随时从GitHub上下载基本的项目。

### 鼓捣代码

乍一看， 响应式编程可能非常棘手，但是过一段时间之后你就会看到你的代码所产生的魔力，让我们现在开始深入代码吧！

这篇文章，我们将面临两个相关的组件，**Observables**和**Subscribers**,

刚开始的主要思想就是有一个可观测的发射字符串***Hello,World***并且终止执行。

	Observable.OnSubscribe observableAction = new Observable.OnSubscribe<String>(){
		public void call(Subscriber<? super String> subscriber){
			subscriber.onNext("Hello,World!");
			subscriber.onCompleted();
		}
	};
	Observable<String> observable = Observable.create(observableAction);
	
下一步就是创建两个订阅者，其中一个将收到的字符串作为参数设置给TextView，另外一个将直接通过Toast打印出来。

	Subscriber<String> textViewSubscriber = new Subscriber<String>(){
		public void onCompleted(){}
		public void onError(Throwable e){}
		public void onNext(String s){
			txtParta.setText(s);
		}
	}
	
	Subscriber<String> toastSubscriber = new Subscriber<String>(){
		public void onCompleted(){}
		public void onError(Throwable e){}
		public void onNext(String s){
			Toast.makeText(context,s,Toast.LENGTH_SHORT).show();
		}
	}
	
最后，我们需要连接这些逻辑，我们需要使用Observable在主线程返回值，并且为他添加订阅者，这样在需要的时候就可以发射对象(这意味着被观察)。

选择适当的线程

	observable.observableOn(AndroidSchedulers.mainThread());
	observable.subscribe(textViewSubscriber);
	observable.subscribe(toastSubscriber);
	
如果你想更深入的了解这些代码，你可以从该[链接](https://github.com/tiwiz/RxAndroidCrunch/releases/tag/Part1)下载压缩包或者[克隆项目](https://github.com/tiwiz/RxAndroidCrunch/)，将会有新的发现。