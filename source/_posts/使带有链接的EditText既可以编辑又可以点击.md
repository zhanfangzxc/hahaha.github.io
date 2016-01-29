title: 使带有链接的EditText既可以编辑又可以点击
date: 2015-12-21 10:44:58
tags:
- Android
- EditText

---
原文地址：[http://blog.danlew.net/2015/12/14/making-edittexts-with-links-both-clickable-and-editable/](http://blog.danlew.net/2015/12/14/making-edittexts-with-links-both-clickable-and-editable/)

最近，我正在实现一个支持链接的Edittext功能，用户可以添加URL，并且在不编辑的情况下可以点击此链接。

当Edittext没有获得焦点的状况下，可以点击链接，然而，当EditText获得焦点的时候，链接不可用，用户可以编辑该输入框。

通过添加一个**OnFocusChangeListener**就可以很简单的实现切换功能，但是用户该如何改变焦点。

长点击是一个方法，另一个当打可以点击Edittext上的其他任何空白区域。就像：

![展示图](http://i.imgur.com/aK3GmtL.png)

不幸的是，由于**LinkMovementMethod**的行为，如果文本的最后一部分是一个链接的话，点击任何地方都会引起链接被打开。

我通过观察发现，这个问题只有在最后的自负是一个链接的时候才会发生，如果我们在文本的最后添加点什么的话？

	//在EditText中创建可以点击的链接
	editText.setMovementMethod(LinkMovementMethod.getInstance());

	Spnnable spnnable = new SpnnableString("http://blog.danlew.net);
	Linkify.addLinks(spnnable,Linkify.WEB_URLS);

	CharSequence text = TextUtils.concat(spnnable,"\u200B");

	editText.setText(text);
	
现在点击空白处将不会引起链接被打开的情况并且能切换到编辑模式。