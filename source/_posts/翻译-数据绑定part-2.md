title: '[翻译]数据绑定part 2'
date: 2015-09-21 10:30:23
categories: Android
tags:
- Android
- 数据绑定

---
上一篇文章，我们使一个基础的twitter客户端运行起来了，但是让我们有点不太理解为什么，所以现在我们就来了解一下数据绑定库都做了什么和如何工作的。

首先要做的就是在我们项目的顶级**build.gradle**文件中的**buildscript**模块中添加一个依赖项:

 
	buildscript {
	    repositories {
	        jcenter()
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:1.3.0'
	        classpath "com.android.databinding:dataBinder:1.0-rc1"
	    }
	}
	 
	allprojects {
	    repositories {
	        jcenter()
	    }
	}

在子项目中的**build.gradle**文件中，我们需要添加一个构建插件:

	apply plugin: 'com.android.application'
	apply plugin: 'com.android.databinding' //就是这一行
	 
	android {
	    compileSdkVersion 23
	    buildToolsVersion "23.0.0"
	 
	    defaultConfig {
	        applicationId "com.stylingandroid.databinding"
	        minSdkVersion 7
	        targetSdkVersion 23
	        versionCode 1
	        versionName "1.0"
	 
	        buildConfigField 'String', 'TWITTER_CONSUMER_KEY', "\"${twitterConsumerKey}\""
	        buildConfigField 'String', 'TWITTER_CONSUMER_SECRET', "\"${twitterConsumerSecret}\""
	        buildConfigField 'String', 'TWITTER_ACCESS_KEY', "\"${twitterAccessKey}\""
	        buildConfigField 'String', 'TWITTER_ACCESS_SECRET', "\"${twitterAccessSecret}\""
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	    packagingOptions {
	        exclude 'META-INF/LICENSE.txt'
	    }
	    lintOptions {
	        disable 'InvalidPackage'
	    }
	}
	 
	dependencies {
	    compile 'com.android.support:support-annotations:23.0.0'
	    compile 'com.android.support:design:23.0.0'
	    compile 'com.android.support:appcompat-v7:23.0.0'
	    compile 'com.android.support:recyclerview-v7:23.0.0'
	    compile 'org.twitter4j:twitter4j-core:4.0.4'
	    compile 'org.twitter4j:twitter4j-async:4.0.4'
	    compile 'com.github.bumptech.glide:glide:3.6.1'
	}
	
值得一提的是这里没有关于数据绑定的编译库添加进去，只是一个简单的构建项的添加。

这可以对于即将发生的事情，给我们一个线索，数据绑定是通过在执行的过程中构建和生成的样板式的代码，通常与绑定数据到视图有关。

在我们的例子中，他解析布局文件和检测被**layout**包装器在**layout/status_item.xml**,基于上面的文件名称，会自动生成一个数据绑定类，命名为**StatusItemBinding**，而且这是一个会在ViewHolder实现类中引用的神秘的类。

然后数据绑定会剔除被<layout>包装的代码，然后移除数据绑定表达式，最后将这些合并成**StatusItemBinding**类，换句话说就是将他转变成一个标准的Android xml文件，并且添加一个java类在运行时实现了绑定。

所以我们现在再看**StatusViewHolder**的时候就会感觉很清晰，而且也明白了是怎么回事:

	public class StatusViewHolder extends RecyclerView.ViewHolder{
		private StatusItemBinding binding;
		
		public StatusViewHolder(View itemView){
			super(itemView);
			binding = DataBinding.bind(itemView);	} 
		
		public void bind(Status status){
			binding.setStatus(status);
		}
	}
	
在构造函数中我们调用**DataBindingUtil.bind(itemView)**将会在构建过程中生成一个StatusItemBinding，然后在**bind(Status status)**方法中我们将会调用一个setter在StatusItemBinding实例中，这个setter是在我们原始布局中声明的类型为**.data.Status**,命名为status的**<data>**节点生成的，当我们调用settter的时候，它将会从原始布局中提取数据绑定表达式执行必要的绑定。

现在我们更好的理解了他是怎么工作的而且为什么当app运行的时候我们能看到tweets的列表

![https://blog.stylingandroid.com/wp-content/uploads/2015/08/Part11-1024x768.png](https://blog.stylingandroid.com/wp-content/uploads/2015/08/Part11-1024x768.png)

非常好，一个简单的布局，有三个items被绑定，并没有节省我们多少时间，只是对于TextView来说省去了找views和设置内容的时间。然而并没有结束，这里还可以为我们做好多关于数据绑定的事情，当我们从Twitter中提取出了image URL之后，然后将他绑定到ImageView中之后，图片却并没有显示出来，我们需要包含一个图像加载库，在下一篇文章中，我们将看到怎样通过数据绑定自动加载图片。

在这篇文章中主要是一些说明，并没有增加额外的代码，所以我们可以使用上一篇文章中的代码。