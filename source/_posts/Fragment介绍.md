title: Fragment介绍
date: 2015-09-22 10:28:32
categories: Fragment
tags:
- Android
- Fragment

---

原文:[http://developer.android.com/intl/zh-cn/guide/components/fragments.html#CommunicatingWithActivity](http://developer.android.com/intl/zh-cn/guide/components/fragments.html#CommunicatingWithActivity)
### Fragment介绍

**Fragment**是可以在Activity中替换的用户行为和用户接口，多个Fragment之间的相互作用是通过**FragmentManager**，而FragmentManager是可以通过**Activity.getFragmentManager()**和**Fragment.getFragmentManager()**.

**Fragment**拥有自己的生命周期，但是他的生命周期同时又依赖Activity的生命周期，Fragment和Activity是密切相关的，不能单独使用，如果Activity的状态是stopped，那么这个Activity中的Fragment也将处于stopped的状态，当所有的Activity被回收的时候，所有的fragments也将会被回收。

### Fragment的生命周期

![Fragment生命周期](http://developer.android.com/images/fragment_lifecycle.png)

- onAttach():当Fragment和Activity建立联系的时候调用
- onCreate(Bundle):初始化创建Fragment的时候调用
- onCreateView(LayoutInflater,ViewGroup,Bundle):创建并返回与之关联的view视图
- onActivityCreated(Bundle):告诉Fragment，Activity已经完成自己的**Activity.onCreate()**方法
- onViewStateRestored(Bundle):告诉Fragment，保存的所有view视图中的状态都已经恢复了
- onStart():使Fragment对用户可见
- onResume():使Fragment开始与用户交互

当这个Fragment不再使用的时候，将会回掉如下方法:

- onPause():Fragment不再与用户交互，因为activity的状态为paused或者Fragment操作被修改了
- onStop():fragment对用户不可见，因为activity的状态为stopped或者activity中的Fragment被修改了
- onDestoryView():允许Fragment清理与视图相关联的资源
- onDestory():最后清理Fragment的状态
- onDetach():当Fragment与activity失去关联的时候调用

### 添加一个用户界面

Fragment经常被用作Activity的一部分，为Fragment提供一个布局，需要实现**onCreate()**方法，当Fragment绘制布局的时候由Android系统调用,你实现这个方法的时候必须返回一个**View**。

注意:如果你的Fragment是**ListFragment**的子类，**onCreateView**方法的默认实现方式会返回一个**ListView**，所以你不需要再去实现这个方法。

	public static class ExampleFragment extends Fragment{
		@Override
		public View onCreateView(LayoutInflater inflater,ViewGroup container,Bundle savedInstanceState){
			return inflater.inflate(R.layout.example_fragment,container,false);
		}
	}

	//这里缺少对改方法的解释
	
### 向Activity添加一个Fragment

Fragment经常被作为activity的一部分添加进去，这里提供两种添加Fragment的方法：

- 在Activity布局文件中声明Fragment

		<?xml version="1.0" encoding="utf-8">
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			android:orientation="horizontal"
			android:layout_width="match_parent"
			android:layout_height="match_parent">
			
			<fragment android:name="com.example.news.ArticleListFragment"
		            android:id="@+id/list"
		            android:layout_weight="1"
		            android:layout_width="0dp"
		            android:layout_height="match_parent" />
		    <fragment android:name="com.example.news.ArticleReaderFragment"
		            android:id="@+id/viewer"
		            android:layout_weight="2"
		            android:layout_width="0dp"
		            android:layout_height="match_parent" />
		</LinearLayout>
		
每个Fragment需要一个唯一的标识符用来当activity重启的时候恢复Fragment，你可以使用他来执行捕获Fragment来执行事务，比如说移除它，这里有三种给Fragment提供id的方法：

- 提供一个**android:id**作为Fragment的唯一标识符
- 提供一个**android:tag**作为一个唯一标识符
- 如果同时提供两者的话，系统会优先使用id

### 或者，向一个已经存在的ViewGroup中自动添加一个Fragment

在acticity运行的任何时候，都可以向activity布局中添加Fragment，具体代码：

	FragmentManager fragmentManager = getFragmentManager();
	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
	ExampleFragment fragment = new ExampleFragment();
	fragmentTransaction.add(R.id.fragment_container,fragment);
	fragmentTransaction.commit();

**add()**方法的第一个参数是一个将会被Fragment替换的ViewGroup，是一个资源id，第二个参数是将要添加的Fragment。

如果你想要通过**FragmentTransaction**改变状态，你需要调用**commit**方法来使改变起作用。

### 管理Fragments

在你的activity中管理Fragment需要使用**FragmentManager**，在activity中调用**getFragmentManager()**来获得**FragmentManager**。

你可以使用**FragmentManager**做以下事情:

- 获取在activity中存在的Fragment，使用**findFragmentById()**或者**findFragmentByTag()**
- 从回退栈中弹出Fragment。使用**popBackStack()**
- 当回退栈改变的时候，注册一个监听器，使用**addOnBackStackChangedListener**

如果想了解更多关于这些方法和其他方法的信息，请参考**FragmentManager**类的文档。

还可以使用**FragmentManager**开启一个事务，然后执行事务，比如添加和移除Fragment。

### 执行Fragment事务

使用Fragment最大的特色就是可以在activity中对他们执行添加、移除、替换以及其他的行为，你还可以保存事务到一个由acticvity管理的后退栈中。

你可以从**FragmentManager**中获取一个**FragmentTransaction**事务

	FragmentManager fragmentManager = getFragmentManager();
	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
	
每个事务是你想同时执行的一组改变，你可以使用**add()**，**remove()**，**replace()**方法，然后把这些事务应用到activity上，需要使用**commit**方法。

在你执行**commit**方法之前，你想要调用**addToBackStack()**把Fragment事务添加到一个后退栈中，这个后退栈是由activity管理的，允许用户按返回键的时候返回前一个Fragment的状态。

	Fragment newFragment = new ExampleFragment();
	FragmentTransaction transaction = getFragmentManager().beginTransaction();

	transaction.replace(R.id.fragment_container,newFragment);
	transaction.addToBackStack(null);
	transaction.commit();

如果你向事务中添加了多个改变，然后调用**addToBackStack()**中的时候，在你调用commit方法之前，所有的改变都会被当成一个单独的事务添加到回退栈中，如果你点击了后退按钮，会把那些所有的改变全都回转过来。

你向**FragmentTransaction**中添加变化的时候，需要注意的是：

- 你必须在最后调用**commit()**
- 当你添加了多个fragments向同一个container中时，你添加的顺序也就是你出现的顺序。

当执行一个移除fragment的事务的时候，如果没有调用**addToBackStack()**方法，那么Fragment将会被回收，当用户操作返回按钮的时候，将不会出现被移除的Fragment；当调用**addToBackStack()**方法的时候，Fragment并没有被移除，而是stopped

> 如果在Fragment事务中想执行一个过度动画雨啊的话，需要在调用**commit()**方法之前调用**setTransition()**。

调用**Commit()**方法并不能理解执行事务，相反的，这个事务实在activity的ui线程(也称主线程)执行的，然而你可以从ui线程调用**executePendingTransaction()**来利己执行提交事务，当然没必要这样做，除非别的线程的工作依赖这个事务的提交。

### 和Activity交互

Fragment可以通过**getActivity()**来获取Activity的实例，而且可以轻松执行任务，比如说查找activity布局中的view。

	View listView = getActivity().findViewById(R.id.list);

在activity中也可以通过**FragmentManager**中的一些方法来获取Fragment的实例，比如**findFragmentById()**或者**findFragmentByTag()**

	ExampleFragment fragment = (ExampleFragment)getFragmentManager().findFragmentById(R.id.example_fragment);

### 向Activity创建一个事件回调

有时候，Fragment需要向Activity分享一个事件，一个很好的做法就是在Fragment中定义一个回调接口然后需要在activity中实现它，当一个activity通过这个接口接收到一个回掉的时候，可以向其它Fragment分享信息。

当一个新闻app在一个activity中存在两个Fragment，一个用来显示新闻列表，一个用来显示文章，所以FragmentA必须告诉FragmentB哪一篇文章被选择了，所以在这个列表中，新闻列表Fragment必须声明一个**OnArticleSelectedListener**接口:

	public static class FragmentA extends ListFragment{
		public interface OnArticleSelectedListener{
			public void onArticleSelected(Uri articleUri);
		}
	}
	
然后在activity中需要实现**OnArticleSelectedListener**接口，并且重写**onArticleSelected()**，将从FragmentA中得到的事件通知FragmentB。为了确保activity实现了这个接口，FragmentA的**onAttach()**方法中必须初始化一个**OnArticleSelectedListener**由activity转换而来的实例。

	public static class FragmentA extends ListFragment{
		OnArticleSelectedListener mListener;
		
		@Override
		puiblic void onAttach(Activity activity){
			super.onAttach(activity);
			try{
				mListener = (OnArticleSelectedListener)activity;
			}catch (ClassCastException e) {
	            throw new ClassCastException(activity.toString() + " must implement OnArticleSelectedListener");
	        }
		}
	}

接下来就是传递事件的逻辑

	public static class FragmentA extends ListFragment {
	    OnArticleSelectedListener mListener;
	    ...
	    @Override
	    public void onListItemClick(ListView l, View v, int position, long id) {
	        // Append the clicked item's row ID with the content provider Uri
	        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
	        // Send the event and Uri to the host activity
	        mListener.onArticleSelected(noteUri);
	    }
	    ...
	}

### 向Action Bar中添加条目

Fragment可以通过实现**onCreateOptionMenu()**方法来向activity的选项菜单中添加菜单项。为了使这个方法接收到调用，所以你必须在**onCreate**方法中调用**setHasOptionsMenu**,表明Fragment将要添加选项到选项菜单中。

### 管理Fragment的生命周期

管理Fragment和管理activity的生命周期很像

![Fragment和Activity的生命周期对比](http://developer.android.com/images/activity_fragment_lifecycle.png)

像activity一样，你可以用**Bundle**来保存一个Fragment的状态，当activity被杀死的时候，你需要保存Fragment的状态，当activity被恢复的时候，可以恢复那些状态，你可以在**onSaveInstanceState()**回调方法中保存Fragment的状态，并且通过**onCreate，onCreateView，onActivityCreated()**方法来恢复之前所保存的状态，更多关于保存状态的信息，请看activities文档。

有一些不同的是，Fragment和Activity是如何存储进各自的后退栈的，对于Activity来说，当一个Activity状态变为stopped的时候，由系统管理的activities后退栈会将当前回收的activity替换进后退栈，但是对于Fragment来说，只有在提交事务之前调用了**addToBackStack()**方法才能进入由activity管理的后退栈。