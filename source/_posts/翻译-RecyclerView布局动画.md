title: '[翻译]RecyclerView布局动画'
date: 2015-09-15 10:34:37
tags:
- Android
- RecyclerView
- layout Animation

---

**翻译原文**：[http://antonioleiva.com/layout-animations-on-recyclerview/](http://antonioleiva.com/layout-animations-on-recyclerview/)
**相关源码**：[https://github.com/antoniolg/MaterializeYourApp](https://github.com/antoniolg/MaterializeYourApp)
> 前言

自从材料设计被提出之后，我惊讶于一些视频中展示的网格填充动画，如果你们还没看到，它由一个对角线动画从上至下，从左至右填充activity，非常漂亮。

![展示图片](http://antonioleiva.com/wp-content/uploads/2015/09/layout_animation_recyclerview.gif)

我尝试了很多方案去获得这个效果，在以后一个解决反感，被很多人提到，那就是使用***RecyclerView::notifyItemInserted()***方法，但是这个方法不能提供太多对于动画顺序的控制，所以看起来并不是个好的解决方案，另外一个，实际上是可行的，只有在必要的时候才在***onBind()***方法中包含元素动画，但是代码是相当微妙和侵入式的(因为我们是在Adapter中添加的动画)，所以要想使它正常运行比较困难。

##开始拯救布局动画吧！

终于，解决方案看起来比预期的更简单，必须承认我之前没怎么使用过布局动画，所以这个选项并不是立刻出现在我的脑海中的，但是当我寻找解决方案的时候，我发现了这个了不起的[Musenkishi发布的gist](https://gist.github.com/Musenkishi/8df1ab549857756098ba)给我提供了一个解决方案，但是问题是RecyclerView默认是不使用布局动画的，但是这段代码可以让它使用GridLayoutAnimations就像使用GridView那样，这是完整的gist:

```
/*
 * Copyright (C) 2014 Freddie (Musenkishi) Lust-Hed
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
 
package com.musenkishi.gists.view;

import android.content.Context;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.view.View;
import android.view.ViewGroup;
import android.view.animation.GridLayoutAnimationController;

/**
 * An extension of RecyclerView, focused more on resembling a GridView.
 * Unlike {@link android.support.v7.widget.RecyclerView}, this view can handle
 * {@code <gridLayoutAnimation>} as long as you provide it a
 * {@link android.support.v7.widget.GridLayoutManager} in
 * {@code setLayoutManager(LayoutManager layout)}.
 *
 * Created by Freddie (Musenkishi) Lust-Hed.
 */
public class GridRecyclerView extends RecyclerView {

    public GridRecyclerView(Context context) {
        super(context);
    }

    public GridRecyclerView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public GridRecyclerView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    public void setLayoutManager(LayoutManager layout) {
        if (layout instanceof GridLayoutManager){
            super.setLayoutManager(layout);
        } else {
            throw new ClassCastException("You should only use a GridLayoutManager with GridRecyclerView.");
        }
    }

    @Override
    protected void attachLayoutAnimationParameters(View child, ViewGroup.LayoutParams params, int index, int count) {

        if (getAdapter() != null && getLayoutManager() instanceof GridLayoutManager){

            GridLayoutAnimationController.AnimationParameters animationParams =
                    (GridLayoutAnimationController.AnimationParameters) params.layoutAnimationParameters;

            if (animationParams == null) {
                animationParams = new GridLayoutAnimationController.AnimationParameters();
                params.layoutAnimationParameters = animationParams;
            }

            int columns = ((GridLayoutManager) getLayoutManager()).getSpanCount();

            animationParams.count = count;
            animationParams.index = index;
            animationParams.columnsCount = columns;
            animationParams.rowsCount = count / columns;

            final int invertedIndex = count - 1 - index;
            animationParams.column = columns - 1 - (invertedIndex % columns);
            animationParams.row = animationParams.rowsCount - 1 - invertedIndex / columns;

        } else {
            super.attachLayoutAnimationParameters(child, params, index, count);
        }
    }
}
view raw
```

> 配置布局动画

布局动画的优点是，我们可以使用xml来定义和使用他，我们只需要定义一个带有相应布局动画的xml.

```
<gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
	android:columnDelay="15%"
	android:rowDelay="15%"
	android:animation="@anim/slide_in_bottom"
	android:animationOrder="normal"
	android:direction="top_to_bottom|left_to_right"/>
```
我们可以定制我们喜欢的动画：

- **columnDelay / rowDelay**:行和列延迟执行动画的时间百分比，这种方法可以用来定义下一行和列中的view可以一个接一个的出现，而不是在同一时间全部出现。
- **animation**:用来展示view的动画，我使用了一个从屏幕底部滑动的动画。
- **animationOrder**:可以是***normal***，***reverse***或者***random***
- **direction**:定义items基于列是如何出现的：top_to_bottom,left_to_right,bottom_to_top,and right_to_left是可选的值。

***下面是显示的动画***:

```
<translate xmlns:android="http://schemas.android.com/apk/res/android"
	android:interpolator="@android:anim/decelerate_interpolator"
	android:fromYDelta="100%p"
	android:toYDelta="0"
	android:duration="android:integer/config_mediumAnimTime"/>
```
> **调整动画时机**

如果你只是这样执行，你将会看到当app打开的时候，布局动画已经执行过了，所以你并不能真正看到动画的执行效果，Lollipop版本之前不能改变什么，因为我们不能有效的知道什么时候进入过渡执行完，但是从Lollipop开始，我们可以使用***onEnterAnimationComplete***来检查，所以，在***onCreate***中，如果SDK版本高于Lollipop，作如下更改：

```
if(Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP){
	setRecyclerAdapter(recyclerView);
}
```
在Lollipop或者更新的SDK版本上，***onEnterAnimationComplete***将会被执行，填充RecyclerView和请求一个新的布局动画的时间:

```
@Override
public void onEnterAnimationComplete(){
	super.onEnterAnimationComplete();
	setRecyclerAdapter(recyclerView);
	recyclerView.scheduleLayoutAnimation();
}
```
> 结论

你可以轻松设置布局动画来创建其他的进入动画，琢磨动画设置看看你能获得什么。
