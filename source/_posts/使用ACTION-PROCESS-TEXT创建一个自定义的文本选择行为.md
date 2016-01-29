title: 使用ACTION_PROCESS_TEXT创建一个自定义的文本选择行为
date: 2016-01-04 09:15:55
tags:
- Android

---
原文地址:[https://medium.com/google-developers/custom-text-selection-actions-with-action-process-text-191f792d2999?linkId=20000023#.ow1rpqsbm](https://medium.com/google-developers/custom-text-selection-actions-with-action-process-text-191f792d2999?linkId=20000023#.ow1rpqsbm)

Android 6.0介绍了一个新的浮动文本选择工具栏，它将带给我们标准的文本选择行为，例如在所选择的文本附近会出现剪切，复制，粘贴等行为，**ACTION_PROCESS_TEXT**使任何的app添加自定义的文本选择工具栏成为可能。

![https://d262ilb51hltx0.cloudfront.net/max/800/1*D4zZzPlBTk5cEN9Qn0-cBA.gif](https://d262ilb51hltx0.cloudfront.net/max/800/1*D4zZzPlBTk5cEN9Qn0-cBA.gif)

像维基百科，谷歌翻译已经利用他可以查找和翻译选中的文本。

你已经可以看到在你的文档中添加这个功能的一些说明文档和博客(总之：使用一个标准的TextView/EditText是可以工作的，但是需要注意的是你的EditText需要设置android:id属性，如果你使用的是**AppCompatActivity**并且想在API 23+设备上使用浮动文本选择工具栏的话，你需要调用**getDelegate().setHandleNativeActionModesEnabled(false)**)

但是实现**ACTION_PROCESS_TEXT**和添加你自己的行为才是这篇文章要讲述的重点:

### 跨越app通讯 -》 Intent Filters

当创建的功能需要跨越app的时候，你的Android Manifest 和intent filters作为一个公共的API连接到每个组件使其他的APP可以查询到。

**ACTION_PROCESS_TEXT**也毫不例外，你将要添加一个intent filter 到你的清淡文件中的一个Activity中。

	<activity android:name="@string/process_text_action_name">
		<intent-filter>
			<action android:name="android.intent.action.PROCESS_TEXT"/>
			<category android:name="android.intent.category.DEFAULT"/>
			<data android:mimetype="text/plain"/>
		</intent-filter>
	</activity>
	
如果你想要多个行动，你需要为每个单独的活动添加上述代码，如果你想只在自己的app中出现所创建的功能，而不在其他应用中出现，那么你可以添加**android:exported="false"**来确保这些行为只会在你的app中出现。

注意你的将要作为文本选择工具栏显示的Activity的android:name要确保他比较短，一个行为动词或者一个可认识的作为你的app的标志。比如，谷歌翻译使用“Translate”作为一个不太常见的行为(有多少人安装了多少翻译软件？),维基百科使用“Search Wikipedia”作为搜索，对很多app来说可能是更常见的行为。

### 获取选择的文本

当你的app设置了intent filter之后，其他的app将会通过选择的文本开启你的activity并且通过文本选择工具栏来选择你所定义的行为。但是并不会增加任何价值，除非你要看所选择的文本。

这也是**EXTRA_PROCESS_TEXT**进来的原因，他是包含在intent中的字符串，代表所选择的文本，不要呗欺骗了——尽管你已经使用了text/plain intent filter,如果你注意到一些样式直接使用CharSequence在你的app中，请不要惊讶。

因此在你的**onCreate()**方法中会看到如下的代码:

	@Override
	protected void onCreate(Bundle savedInstanceState){
		super.onCreate(saved);
		setContentView(R.layout.process_text_main);
		CharSequence text = getIntent().getCharSequenceExtra(Intent.EXTRA_PROCESS_TEXT);
		
	}
	
如果你使用的是android:launchMode="singleTop"的话，会有一个警告,然后你可能想要在**onNewIntent()**中执行text，常见的操作就是当你创建的时候在onCreate()和onNewIntent()中调用handleIntent()方法。

这就是使用ACTION_PROCESS_TEXT作为你的应用程序的入口所要做的操作，至于之后用它做什么，全部由你决定。

### 返回结果

还有一个额外的包含在**ACTION_PROCESS_TEXT**的intent通过:EXTRA_PROCESS_TEXT_READONLY,这个布尔值可以表示你刚刚接收到的所选择的文本是可以被用户编辑的。

代码如下：

	boolean readonly = getIntent()
		.getBooleanExtra(Intent.EXTRA_PROCESS_TEXT_READONLY,false);

你可以使用它作为一个提示提供一种返回修改过的内容给发送的应用程序的能力，从而替换选择的文本。这个可以作为你的activity工作，但是开始于startActivityForResult(),将会通过在Activity结束的时候调用setResult()返回一个结果。

	Intent intent = new Intent();
	intent.putExtra(Intent.EXTRA_PROCESS_TEXT,replacementText);
	setResult(RESULT_OK,intent);
	
你可以想象一个按钮被替换，在Activity返回的时候调用setResult()。

### 常见问题

在开始编写程序之前，这里有一些关于ACTION_PROCESS_TEXT的常见问题

问题:如何通过ACTION_PROCESS_TEXT触发服务

回答:不能直接触发，系统只会查找包含正确intent filter的Activity,这并不意味着你不能通过你的Activity来开启一项服务使用Theme.Translucent.NoTitleBar或者Theme.NoDisplay的主题，但是你要确保你接收到他们的行为的时候有一个用户可见的提示，例如一个通知，或者一个吐司。

问题:我能仅对特定类型的文本触发它吗？

回答:不能，你的选项将会出现在任何人任何时间选择文本的时候，当然，用户不会选择一个翻译，除非他们需要翻译，但是你要小心的防守代码，因为你不知道你将会收到什么类型的文本。

问题:所有的app都需要实现ACTION_PROCESS_TEXT吗？

回答:的确是没必要的，不是每个应用程序都要实现ACTION
_PTOCESS_TEXT，但是要确保我们所实现的任何行为都是普遍的并且对安装应用的用户来说是真正有用的。

### 相关文章

[http://android-developers.blogspot.com/2015/10/in-app-translations-in-android.html](http://android-developers.blogspot.com/2015/10/in-app-translations-in-android.html)