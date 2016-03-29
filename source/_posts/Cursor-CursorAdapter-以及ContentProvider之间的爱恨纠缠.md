title: Cursor CursorAdapter 以及ContentProvider之间的爱恨纠缠
date: 2016-03-29 16:38:21
tags:
- Android
- CursorAdapter
- Cursor

---

### Cursor CursorAdapter 以及 ContentProvider三者之间的关系

#### Cursor和ContentProvider的关系

在写ContentProvider的时候，我们会在query中添加如下代码

	 if(cursor!=null){
	 	cursor.setNotificationUri(getContext().getContentResolver(),url);
	 }
	 
现在通过源码，我们看看setNotificationUri方法做了什么

	  public void setNotificationUri(ContentResolver cr, Uri notifyUri, int userHandle) {
	        synchronized (mSelfObserverLock) {
	            mNotifyUri = notifyUri;
	            mContentResolver = cr;
	            if (mSelfObserver != null) {
	            //如果mSelfObserver不为空的情况下，就取消注册
	           mContentResolver.unregisterContentObserver(mSelfObserver);
	            }
	            //然后创建个新的SelfContentObserver
	            mSelfObserver = new SelfContentObserver(this);
	            //重新注册
	  mContentResolver.registerContentObserver(mNotifyUri, true, mSelfObserver, userHandle);
	            mSelfObserverRegistered = true;
	        }
	    }

在ContentProvider中的delete、update、insert方法中，我们会添加如下代码：

	if(affectedRow > 0){
		getContext().getContentResolver().notifyChange(uri,null);
	}

notifyChange方法主要是用来通知所有的观察者数据库改变了，默认情况下CursorAdapter实例是可以收到这个通知的。
此时当数据库改变的时候，AbstractCursor中的mSelfObserver中的onChange方法就会执行，下面看一下源码:

	 @Override
        public void onChange(boolean selfChange) {
            AbstractCursor cursor = mCursor.get();
            if (cursor != null) {
                cursor.onChange(false);
            }
        }
        
接着就会执行Cursor的onChange方法：

	protected void onChange(boolean selfChange) {
        synchronized (mSelfObserverLock) {
        //此处调用了dispatchChange方法
            mContentObservable.dispatchChange(selfChange, null);
            if (mNotifyUri != null && selfChange) {
                mContentResolver.notifyChange(mNotifyUri, mSelfObserver);
            }
        }
    }
    
在ContentObservable方法中的dispatchChange方法中调用了所有的ContentObserver的dispatchChange方法，其中就包含在CursorAdapter中注册的ChangeObserver，看ContentObserver的源码可以看到，此处接着会去调用onChange方法，也就是会去调用ChangeObserver的onChange方法。也就是下面所说的onContentChanged()方法。

#### Cursor和CursorAdapter的关系

在CursorAdapter的构造函数中需要传入一个Cursor对象，然后构造函数会调用init方法，该方法的源码如下：

	void init(Context context, Cursor c, int flags) {
        if ((flags & FLAG_AUTO_REQUERY) == FLAG_AUTO_REQUERY) {
            flags |= FLAG_REGISTER_CONTENT_OBSERVER;
            mAutoRequery = true;
        } else {
            mAutoRequery = false;
        }
        boolean cursorPresent = c != null;
        mCursor = c;
        mDataValid = cursorPresent;
        mContext = context;
        mRowIDColumn = cursorPresent ? c.getColumnIndexOrThrow("_id") : -1;
        if ((flags & FLAG_REGISTER_CONTENT_OBSERVER) == FLAG_REGISTER_CONTENT_OBSERVER) {
       //此处创建两个ContentObserver
            mChangeObserver = new ChangeObserver();
            mDataSetObserver = new MyDataSetObserver();
        } else {
            mChangeObserver = null;
            mDataSetObserver = null;
        }

        if (cursorPresent) {
            if (mChangeObserver != null) c.registerContentObserver(mChangeObserver);
            if (mDataSetObserver != null) c.registerDataSetObserver(mDataSetObserver);
        }
    }
    
init方法中会创建两个ContentObserver,一个是ChangeObserver，一个是DataSetObserver,并且通过Cursor注册这两个观察者，如果获取到改变的时候，就会分别调用这两个Observer的onChange方法:

	 private class ChangeObserver extends ContentObserver {
        public ChangeObserver() {
            super(new Handler());
        }

        @Override
        public boolean deliverSelfNotifications() {
            return true;
        }

        @Override
        public void onChange(boolean selfChange) {
            onContentChanged();
        }
    }
    
 也就是执行onContentChanged()方法，现在来看看这个方法的源码：
 
 	protected void onContentChanged() {
 		//如果设置了自动重新查询的话
        if (mAutoRequery && mCursor != null && !mCursor.isClosed()) {
            if (false) Log.v("Cursor", "Auto requerying " + mCursor + " due to update");
            //此时Cursor就会重新查询数据
            mDataValid = mCursor.requery();
        }
    }
    
此处虽然是重新查询了，但是没有notifyDataSetChange，所以list怎么刷新的呢，接着看一下Cursor中的requery方法：

	public boolean requery() {  
        if (mSelfObserver != null && mSelfObserverRegistered == false) {  
            mContentResolver.registerContentObserver(mNotifyUri, true, mSelfObserver);  
            mSelfObserverRegistered = true;  
        }  
        //此处DataSetObservable收到变化的时候，就会调用他自己的onChanged方法
        mDataSetObservable.notifyChanged();  
        return true;  
}  

在他的onChanged方法中：

	 private class MyDataSetObserver extends DataSetObserver {
        @Override
        public void onChanged() {
            mDataValid = true;
            //就是这个了
            notifyDataSetChanged();
        }

        @Override
        public void onInvalidated() {
            mDataValid = false;
            notifyDataSetInvalidated();
        }
    }
