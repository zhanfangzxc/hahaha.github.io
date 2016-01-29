title: RxAndroid 第一部分
date: 2016-01-25 10:56:07
tags:
- Android
- RxAndroid

---
原文地址:[https://medium.com/crunching-rxandroid/crunching-rxandroid-part-1-4ac7b7123238#.vo9v1p16d](https://medium.com/crunching-rxandroid/crunching-rxandroid-part-1-4ac7b7123238#.vo9v1p16d)

在上一篇文章中我们了解了一些基本的Rxjava的东西，接下来我们准备更深入一点。

首先，我们今天的主要目的就是当我们已经更多的了解了上篇文章的内容之后就开始缩短代码，让我们开始创建一些东西并且只把输入作为TextView的一个文本：

	Action1<String> textViewOnNextAction = new Action1<String>(){
		@Override
		public void call(String s){
			txtPart1.setText(s);
		}
	}
	
分析上面的代码，我们可以意识到我们并没有创建一个完整的并且合格的订阅者。我们只实现了当成功的时候将会发生的一些事情，避免了与其他事件相关的我们不需要的代码。

有时候我们并不仅仅想为某个View设置一个标签，我们真正需要的是将字符串转换成大写的，因为我们想向全世界呼喊，我们可以使用一个函数，或多或少如下面这样

	Func1<String,String> toUpperCaseMap = new Func1<String,String>(){
		@Override
		public String call(){
			return s.toUpperCase();
		}
	}
	
这时候我们就只需要创建一个新的Observable并且关联所有的逻辑，我们一次只需要发送一个单独的字符串，我们可以使用一个快捷健创建一个可观察到的静态的方法。

	Observable<String> singleObservable = Observable.just("Hello,World!");

现在我们需要将观察者,方法和订阅者串联起来，我们将会看到下面这些：

	singleObservable.observableOn(AndroidSchedulers.mainThread())
		.map(toUpperCaseMap)
		.subscribe(textViewOnNextAction);
		
你能察觉到这里的新闻吗，我说的就是**map**操作符。

### 操作符

再这个例子中，我们说明了第一个操作符，在这个特殊的例子中，一个从字符串到字符串的方法将使我们可以再将他传递给最终的订阅者之前对从观察者的任务进行计算。RxJava提供了许多不同的操作符，我们可以使用他来将任何东西转换成我们需要的数据。

### 使用数组和集合

我们看到了如何发射一个单独的条目，但是当我们想处理一组条目，并且一个接一个的发射他们该怎么做？我们从最简单的情况开始，我们想使用一个字符串数组并且将他们用Toast打印出来。

让我们定义一个订阅者只是用来对任意输出的字符串进行输出，所以我们必须在队列结束时运行：

	Action1<String> toastOnNextAction = new Action1<String>(){
		@Override
		public void call(String s){
			Toast.makeText(context,s,);
		}
	}
	
正如我们看到的一样，这里没有什么新的东西，所以我们可以跳过直接去创建可观察对象，和我们之前例子中所做的一样，但是这里我们可以使用数组来一个接一个的发送条目。

	Observable<String> oneByOneObservable = Observable.from(manyWords);
	
使用**from()**方法我们可以按顺序发送数组中的左右条目，我们不会太多的关系是怎么通过他们循环的，太棒了！

让我们接下来面对一个更复杂一点的例子，我们需要在一次单独的发射中一次发射一个完整的集合,然后处理每一个元素，首先，我们需要发射整个集合，从第一个例子中我们知道该怎么做，从工厂方法中创建一个单独的Observable:

	Observable.just(manyWordList);
	
然后我们需要一个可观察的东西来返回另一个可观察的东西，或许我们需要一个操作符

### 介绍flatMap

我们看到map操作符可以将一些东西转换成另一些东西，但这不是他唯一能派上用场的地方，在这个例子中我们可以将一个Observable转换成另一个Observable，实现起来非常简单，我们可以再眨眼之间就能看到结果

	Func1<List<String>,Observable<String>> getUrls = new Func1<List<String>,Observable<String>>(){
		@Override
		public Observable<String> call(List<String> strings){
			return Observable.from(strings);
		}
	}
	
现在我们希望显示一个良好的格式化后的消息，我们可以使用**reduce**运算符,一旦Observable完成大蛇，他将会把获取到的字符串当作一个方法的输入进行合并

	Func2<String,String,String> mergeRoutine = new Func2<String,String,String>(){
		@Override
		public String call(String s,String s1){
			return String.format("%s, %s",s,s1);
		}
	}
	
最后我们将所有的东西都连接起来，看看会看到什么神奇的事情发生！

	Observable.just(manyWordList)
		.observableOn(AndroidSchedulers.mainThread())
		.flatMap(getUrls)
		.reduce(mergeRoutine)
		.subscribe(toastOnNextAction);