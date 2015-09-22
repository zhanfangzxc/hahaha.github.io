title: '[翻译]数据绑定part 1'
date: 2015-09-15 13:14:20
categories: Android
tags:
- Android
- Data Binding

---

**翻译原文**：[https://blog.stylingandroid.com/data-binding-part-1/](https://blog.stylingandroid.com/data-binding-part-1/)
**相关源码**：[https://github.com/StylingAndroid/DataBinding/tree/Part1](https://github.com/StylingAndroid/DataBinding/tree/Part1)
### 前言

谷歌2015年I/O大会中揭晓了许多开源库和工具，其中一个全新的开源库就是***数据绑定***，在这个系列的文章中我们将一起探索一下这个开源库所提供的强大的功能。

在开始之前值得一提的是，在写作这篇文章的时候，数据绑定开源库仍然处于测试阶段，所以在完整版发布之前，一些Api的调用还是有可能改变。

***Data Binding***提供了一种连接布局和数据源并将数据显示在布局上的机制，这个示例代码使用Twitter Api来获取数据，来实现一个简单的Twitter客户端，这里不打算提供Twitter API和app的基本设计的指导，使用Twitter4j库来检索你的twitter时间轴中的最后50条数据现在在RecyclerView中，所有的代码都是公开的，可以随时在浏览器中检索学习，这系列文章中比较有趣的地方是如何将数据绑定到视图。

让我们先看一下数据模型:


	public class Status{
		private final String name;
		private final String screenName;
		private final String text;
		private final String imageUrl;
		
		public Status(@NonNull String name,@NonNull String screenName,@NonNull String text,@NonNull String imageUrl){
			this.name = name;
			this.screenName = screenName;
			this.text = text;
			this.imageUrl = imageUrl;
		}
		
		public String getName(){
			return name;
		}
		
		public String getScreenName(){
			return screenName;
		}
		
		public String getText(){
			return text;
		}
		
		public String getImageUrl(){
			return imageUrl;
		}
	}


这仅仅是用来绑定到RecyclerView的每一个条目中用来显示给用户的值，这些数据从twitter4j.Status对象中创建，由Twitter4j检索，所以我们需要有一个类用来将twitter4j.status转换为我们创建的Status:


	public class StatusMarshaller{
		public List<Status> marshall(List<twitter4j.Status> statues){
			for(twitter4j.Status status:statuses){
				newStatues.add(marshall(status));
			}
			return newStatuses;
		}
		
		private Status marshall(twitter4j.Status status){
			User user = status.getUser();
			return new Status(user.getName(),user.getScreenName(),status.getText(),user.getBiggerProfileImageURL());
		}
	}


这里没有什么难懂的代码，只是纯粹的java代码，没有关于数据绑定的东西。

值得一提的是，我们可以直接使用twitter4j.Status对象来进行数据绑定，在最初的例子中会更有效率。数据绑定库是一个MVVM模型,Model是twitter4j.Status,View就是我们的布局，ModelView是我们创建的Status对象，ViewModel是为了更容易使用而设计的数据视图，Model和ViewModel几乎是一样的，而且也很难看出有什么有点，但是当我们深入研究的时候，就能发现其中蕴含的诸多优点。

接下来就是创建RecyclerView的适配器：


	public class StatusAdapter extends RecyclerView.Adapter<StatusViewHolder>{
		  private final List<Status> statuses;
		  private final StatusMarshaller marshaller;
		  
		  public static StatusAdapter newInstance(){
		  		List<Status> statuses = new ArrayList<>();
		  		StatusMarshaller = new StatusMarshaller();
		  		return new StatusAdapter(statuses,marshaller);
		  }
		  
		  StatusAdapter(List<Status> statuses,StatusMarshaller marshaller){
		  		this.statuses = statuses;
		  		this.marshaller = marshaller;
		  } 
		  
		  @Override
		  public StatusViewHolder onCreateViewHolder(ViewGroup parent,int viewType){
		  		Context contenxt = parent.getContext;
		  		LayoutInflater inflater = LayoutInflater.from(context);
		  		View statusContainer = inflater.inflate(R.layout.status_item,parent,false);
		  		return new StatusViewHolder(statusContainer);
		  }  
		  
		  @Override
		  public void onBindViewHolder(StatusViewHolder holder,int position){
		  		Status status = stautses.get(position);
		  		holder.bind(status);
		  }
		  
		  @Override
		  public int getItemCount(){
		  		return statuses.size();
		  }
		  
		  public void setStatuses(List<twitter4j.Status> statuses){
		  		this.statuses.clear();
		  		this.statuses.addAll(marshaller.marshall(statuses));
		  		notifyDataChanged();
		  }
	}

现在我们进入安卓领域，但是也没有什么特别的，上面的代码只是一个标准的RecyclerView适配器的实现，并且不带数据绑定，这些将在StatusViewHolder中做，这做的唯一事情就是针对MVVM模式就是通过setStatuses()方法从twitter4j.Status整理出.data.Status。现在我们开始关注RecyclerView的item布局:


	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">

		<data>
			<variable
				name="Status"
				type="com.stylingandroid.databinding.data.Status"/>
		</data>
		<RelativeLayout
	    android:id="@+id/status_container"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	 
	    <ImageView
	      android:id="@+id/status_avatar"
	      android:layout_width="64dp"
	      android:layout_height="64dp"
	      android:layout_alignParentLeft="true"
	      android:layout_alignParentStart="true"
	      android:layout_alignParentTop="true"
	      android:layout_margin="8dip"
	      android:contentDescription="@null" />
	 
	    <TextView
	      android:id="@+id/status_name"
	      style="@style/Status.Name"
	      android:layout_width="wrap_content"
	      android:layout_height="wrap_content"
	      android:layout_alignParentTop="true"
	      android:layout_marginTop="8dip"
	      android:layout_toEndOf="@id/status_avatar"
	      android:layout_toRightOf="@id/status_avatar"
	      android:text="@{status.name}" />
	 
	    <TextView
	      android:id="@+id/status_screen_name"
	      style="@style/Status.ScreenName"
	      android:layout_width="wrap_content"
	      android:layout_height="wrap_content"
	      android:layout_alignBaseline="@id/status_name"
	      android:layout_marginLeft="4dip"
	      android:layout_marginStart="4dip"
	      android:layout_toEndOf="@id/status_name"
	      android:layout_toRightOf="@id/status_name"
	      android:text="@{&quot;@&quot; + status.screenName}" />
	 
	    <TextView
	      android:id="@+id/status_text"
	      style="@style/Status.Text"
	      android:layout_width="match_parent"
	      android:layout_height="wrap_content"
	      android:layout_alignLeft="@id/status_name"
	      android:layout_alignParentEnd="true"
	      android:layout_alignParentRight="true"
	      android:layout_alignStart="@id/status_name"
	      android:layout_below="@id/status_name"
	      android:maxLines="2"
	      android:singleLine="false"
	      android:text="@{status.text}" />
	 
	  </RelativeLayout>
	</layout>

以上就是数据绑定库是如何工作的，RelativeLayout只是一个普通的layout，但是其中参杂了一些奇怪的东西，现在我们就来解释这些东西，但是这个RelativeLayout被layout布局包裹着，这就是定义数据绑定的，和RelativeLayout一样，<layout>包含一个***data***部分，用来定义用于绑定到layout中的那些数据对象，在这个例子中，我们生命了一个data.Status,命名为status，现在就可以在layout中引用了。

上面的layout中还有奇怪的android:text属性，这些是从data.Status对象中getters中获取值，***@{}***是数据绑定的表达式，status.name和Status.getName()的性质是一样的， 这就是数据绑定的工作原理，但是接下来我们将会看到一个比较恐怖的事情.

现在有一个稍微复杂点的***@{&quot;@&quot; + status.screenName}***，尽管看起来比较复杂，但是都是一些简单的字符串连接到Twitter name上而已，现在我们需要把@放在需要被转义成不是xml布局代码的地方，但是本质上都是***"@"+status.getScreenName()***,在java中，这并不能代表我们用这个表达式能够做的更多。

现在我们需要定义StatusViewHolder:


	public class StatusViewHolder extends RecyclerView.ViewHolder{
		private StatusItemBinding binding;
		
		public StatusViewHolder(View itemView){
			super(itemView);
			binding = DataBindingUtil.bind(itemView);
		}
		
		public void bind(Status status){
			binding.setStatus(status);
		}
	}


比正常的ViewHolder要简单很多，我们需要在构造函数中找到父布局中的所有子视图，并且在bind()方法中给每个子视图设置属性，我们首先需要注意的就是我们需要从***DataBindingUtil.bind()***方法中获取我们要使用的***StatusItemBinding***，我们不需要创建这个类，因为数据绑定库会给我们自动创建出来。

我们将在下一篇文章中更加深入的讨论数据绑定，这里只是一个简单的工作流程。