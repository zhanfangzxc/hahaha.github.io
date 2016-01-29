title: Webview操作整理
date: 2015-11-17 16:08:26
tags:
- Android
- Webview

---
如果你想在自己的项目中添加加载网页的功能的话，就需要使用Webview。

### 添加WebView

	<?xml version="1.0" encoding="utf-8"?>
	<WebView  xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/webview"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent"
	/>
	
- 加载网页

		WebView myWebView = (WebView) findViewById(R.id.webview);
		myWebView.loadUrl("http://www.example.com");
		
- 添加权限

		<manifest ... >
	    <uses-permission android:name="android.permission.INTERNET" />
	    ...
		</manifest>
		
### 在WebView中使用JavaScript

JavaScript默认在Webview中是不可用的，但是你可以通过**WebSettings**让他变为可用，具体代码:

	WebView myWebView = (WebView) findViewById(R.id.webview);
	WebSettings webSettings = myWebView.getSettings();
	webSettings.setJavaScriptEnabled(true);
	
WebSettings中还存在很多其他的有用的设置。

### 将javaScript代码绑定到Android代码中

首先，在你的项目中添加如下类：

	public class WebAppInterface{
		Context context;
		WebAppInterface(Context c){
			context = c;
		}
		
		@JavaScriptInterface
		public void showToast(){
			 Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
		}
	}
	
在targetSdkVersion为17或者更高的时候，需要添加**@JavaScriptInterface**到类中的每一个方法上。

绑定代码：

	WebView webView = (WebView) findViewById(R.id.webview);
	webView.addJavascriptInterface(new WebAppInterface(this),"Android");
	
此时的html代码中必须要包含以下代码：

	<input typpe="button" value="Say hello" onClick="showAndroidToast('Hello Android!')"

	<script type="text/javascript">
		function showAndroidToast(toast){
			Android.showToast(toast);
		}
	</script>
	
### 处理Page导航

如果用户要在当前WebView中打开链接，可以使用**setWebViewClient()**为Webview设置**WebviewClient**，比如：

	WebView myWebView = (WebView)findViewById(R.id.webview);
	myWebView.setWebViewClient(new WebViewClient());

现在，所有的用户点击的跳转都会在Webview中加载。

如果点击链接的时候，需要更多的控制，那么就需要创建自己的**WebviewClient**,然后覆盖**shouldOverrideUrlLoading()**方法，代码如下：

	private class MyWebviewClient extends WebViewClient{
		@Override
		public boolean shouldOverrideUrlLoading(WebView view,String url){
			if(Uri.parse(url).getHost().equals("www.example.com")){
				//让WebView加载网页
				return false;
			}
			//不加载网页，跳到另外一Activity
			Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
	        startActivity(intent);
	        return true;
		}
	}
	
最后：

	WebView myWebView = (WebView) findViewById(R.id.webview);
	myWebView.setWebViewClient(new MyWebViewClient());
	
### 操纵网页浏览历史

使用设备上的back健来操纵网页的前进后退:

	@Override
	public boolean onKeyDown(int keyCode,KeyEvent event){
		if((keyCode == KeyEvent.KEYCODE_BACK)&&myWebView.canGoBack())
		{
			myWebView.goBack();
			return true;
		}
		return super.onKeyDown(keyCode,event);
	}

### 实现点击WebView中的图片预览的效果

代码：

	//在Html中添加Javascript代码
	private void addImageClickListner() {
	        mWebView.loadUrl("javascript:(function(){" +
	                "var objs = document.getElementsByTagName(\"img\"); " +
	                "for(var i=0;i<objs.length;i++)  " +
	                "{"
	                + "    objs[i].onclick=function()  " +
	                "    {  "
	                + "        window.imagelistner.openImage(this.src);  " +
	                "    }  " +
	                "}" +
	                "})()");
	    }
	    
	    //创建自定义的WebviewClient
	    
	    private class MyWebViewClient extends WebViewClient {
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            if (url.endsWith(".jpg") || url.endsWith(".png") || url.endsWith(".gif")) {
                return true;
            } else {
                return false;
            }
        }

        @Override
        public void onPageFinished(WebView view, String url) {

            view.getSettings().setJavaScriptEnabled(true);
            mStatusLayout.showContentView(contentLayout);
            super.onPageFinished(view, url);
            addImageClickListner();

        }

        @Override
        public void onPageStarted(WebView view, String url, Bitmap favicon) {
            view.getSettings().setJavaScriptEnabled(true);

            super.onPageStarted(view, url, favicon);
        }

        @Override
        public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {

            super.onReceivedError(view, errorCode, description, failingUrl);

        }
    }
    
      public class JavascriptInterface {

        private Context context;

        public JavascriptInterface(Context context) {
            this.context = context;
        }

        @android.webkit.JavascriptInterface
        public void openImage(String img) {
           //预览图片的代码
        }
    }
    
    //最后一步
      mWebView.addJavascriptInterface(new JavascriptInterface(this), "imagelistner");
      
### WebView拦截替换网络请求数据

如果想让WebView在处理网络请求的时候将某些请求拦截替换成某些特殊的资源，就需要用到下面的方法：

**shouldInterceptRequest**

示例代码：

	webview.setWebViewClient(new WebViewClient(){
		@Override
		public WebResourceReponse shouldInterceptRequest(Webview view,String url){
			WebResourceResponse reponse = null;
			if(url.contains("logo")){
				try {
	              InputStream localCopy = getAssets().open("droidyue.png");
	              response = new WebResourceResponse("image/png", "UTF-8", localCopy);
	          	} catch (IOException e) {
	              e.printStackTrace();
	          	}    
			}    
		}
	})
	
WevResourceResponse需要设定耽搁属性，MIME类型，数据编码，数据流形式。

### 常见问题汇总

- Android 4.4中播放Html5的音频或视频，出现的**E/chromium: [ERROR:in_process_view_renderer.cc(189)] Failed to request GL process. Deadlock likely: 0**问题

解决方案:**mWebView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);**

参考链接:[https://code.google.com/p/android/issues/detail?id=63754](https://code.google.com/p/android/issues/detail?id=63754)

### 参考文章

[http://droidyue.com/blog/2014/11/23/block-web-resource-in-webview/](http://droidyue.com/blog/2014/11/23/block-web-resource-in-webview/)

**WebView问题总结**:[http://blog.csdn.net/t12x3456/article/details/13769731](http://blog.csdn.net/t12x3456/article/details/13769731)

**WebView使用小结**:[http://blog.csdn.net/janice0529/article/details/41318755](http://blog.csdn.net/janice0529/article/details/41318755)

