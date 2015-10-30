title: Touch事件的传递机制
date: 2015-09-24 15:57:16
categories: Android
tags:
- Acdroid
- Touch事件

---

图示：

![Touch事件传递](https://github.com/zhanfangzxc/zhanfangzxc.github.io/blob/source/source/_posts/Touch%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92.jpg?raw=true)

这篇文章为收集整理别人的内容，所以非常感谢那些默默付出的大牛们，正是有你们的付出，才让我不断的进步，感谢。

以下图片中的黑色箭头为**ACTION_DOWN**事件的分发顺序，绿色箭头是**ACTION_UP**和**ACTION_MOVE**的事件分发顺序。

#### ViewGroup和View都不消耗事件，最终交给Activity消耗

![Activity消耗事件](http://fangjie.info/wp-content/uploads/2015/09/view1-1024x629.png)

实际开发中我们经常会**setOnTouchListener**，这个会在**onTouchEvent**之前调用，如果这个方法返回false的话，那么事件会传递到**onTouchEvent**;如果返回true，相当于拦截了事件。

#### Touch的区域是在ViewGroup之内，View之外

![Touch ViewGroup](http://fangjie.info/wp-content/uploads/2015/09/view2.png)

#### View消耗事件

![View消耗事件](http://fangjie.info/wp-content/uploads/2015/09/view3.png)

#### ViewGroup消耗事件，但经过View传递

![ViewGroup消耗事件](http://fangjie.info/wp-content/uploads/2015/09/view4.png)      

#### ViewGroup消耗事件，但是不经过View传递

![ViewGroup](http://fangjie.info/wp-content/uploads/2015/09/view5.png)

#### 参考文章

图解View的事件分发机制:[http://fangjie.info/%E5%9B%BE%E8%A7%A3view%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/?utm_source=tuicool](http://fangjie.info/%E5%9B%BE%E8%A7%A3view%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/?utm_source=tuicool)