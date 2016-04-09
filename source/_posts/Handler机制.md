title: Handler机制
date: 2016-03-30 10:33:46
tags:
- Android
- Handler

---

#### 创建Handler

如果在子线程中尝试进行ui操作的话，程序就会崩溃，此时就需要用Handler来实现线程间的通信。

所以代码如下：

	Handler mHander = new Handler();
	
如果把这句代码按如下方式写：

	new Thread(){
	
		@Override
		public void run(){
		//在子线程中创建Handler
			mHandler = new Handler();
		}
	}.run();

此时就会报错，原因是不能在没有调用Looper.prepare()的情况下创建Handler。

现在我们将上面的代码改成如下代码：

	new Thread(){
		@Override
		public void run(){
			Looper.prepare();
			mHandler = new Handler();
		}
	}.run();
	
现在看一下prepare方法做了什么？

 	 private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
从这里我们看到他创建了一个新的Looper，并且我们看到了在一个线程中只能创建一个Looper。

可是转念一想为什么在子线程中创建Handler需要调用呢，我们看一下Handler的构造函数：

	 public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
		//而这里是什么意思呢？
        mLooper = Looper.myLooper();
        if (mLooper == null) {
        //这里就是我们报错的地方
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
    
结合上面的源码，我们来看看Looper.myLopper函数的作用：

	public static @Nullable Looper myLooper() {
		//从ThreadLocal中获取到一个Looper对象
        return sThreadLocal.get();
    }
    
ThreadLocal是用来存储Looper对象的集合，每个线程只能存在一个Looper，所以myLooper的作用就是返回一个Looper，但是Looper对象是在何时给设置进去的呢，这个就是我们上面说的prepare的作用了。

相反的，我们又会问，那么ui线程的Looper是什么时候初始化的呢，其实在系统启动的时候，就会在ActivityThread的main方法中初始化一个主线程的Looper。

	//这个方法看着会不会感觉格外的熟悉
	Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    thread.attach(false);

    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    if (false) {
        Looper.myLooper().setMessageLogging(new
        LogPrinter(Log.DEBUG, "ActivityThread"));
    }

    // End of event ActivityThreadMain.
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    Looper.loop();
    
从上面我们我们看到了相似的代码Looper.prepareMainLooper();

	public static void prepareMainLooper() {
	//此处也是先调用了prepare方法创建了一个Looper，然后设置进ThreadLocal中，所以之后再创建Handler的时候，就不需要调用Looper的prepare方法了。
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
    
#### 创建Message用来发送消息

代码如下：

	Message message = new Message();
	//Message的这些变量都可以用来设置值
	message.what = 1;
	message.arg1 = 1;
	message.arg2 = 2;
	message.obj=""
	mHandler.sendMessage(message);
	
我们看到发送消息是通过Handler来发送的，那么消息究竟是被发送到哪里了呢？
		
	//一步步往下看，发下具体实现在这里
	 public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
	 	//这个从字面上来看应该是一个消息队列
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        //uptimeMillis表示消息发送的时间，这个值等于系统开机到当前时间的毫秒数再加上延迟时间
        return enqueueMessage(queue, msg, uptimeMillis);
    }
    
我们看到msg又被传到了enqueueMessage方法中，这个方法具体做了什么呢：

我们去看Looper类的源码，发现在Looper的构造函数中创建了一个MessageQueue对象，这个对象就是用来维护消息的消息队列，接着看上面提到的这个方法：

	//消息的入队操作
	final boolean enqueueMessage(Message msg, long when) {  
    if (msg.when != 0) {  
        throw new AndroidRuntimeException(msg + " This message is already in use.");  
    }  
    if (msg.target == null && !mQuitAllowed) {  
        throw new RuntimeException("Main thread not allowed to quit");  
    }  
    synchronized (this) {  
        if (mQuiting) {  
            RuntimeException e = new RuntimeException(msg.target + " sending message to a Handler on a dead thread");  
            Log.w("MessageQueue", e.getMessage(), e);  
            return false;  
        } else if (msg.target == null) {  
            mQuiting = true;  
        }  
        msg.when = when;  
        //mMessage表示当前待处理的消息
        Message p = mMessages;  
        if (p == null || when == 0 || when < p.when) {  
        //如果传进来的消息的时间<待处理的消息时间
        //那么就把待处理消息放到该消息之后
        //然后把该消息给设置为当前待处理消息
            msg.next = p;  
            mMessages = msg;  
            this.notify();  
        } else { 
        	//相反的就把该消息加到待处理消息之后 
            Message prev = null;  
            while (p != null && p.when <= when) {  
                prev = p;  
                p = p.next;  
            }  
            msg.next = prev.next;  
            prev.next = msg;  
            this.notify();  
        }  
    }  
    return true;  
	}  

以上的就是插入消息队列的逻辑，具体的看代码的注释。

然后循环处理消息的代码在Looper类的loop方法中，源码如下：

	 public static void loop() {
	 //首先获取Looper对象
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
        //对消息队列中的消息无限循环
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
			//此处就是将待处理的消息分发给Handler去处理
			//
            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }
			//回收消息
            msg.recycleUnchecked();
        }
    }
    
上面的msg.target实际上就是一个Handler对象，所以具体处理消息的功能还是得依靠Handler，也就是我们经常重写的handleMessage。

	public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            
            handleMessage(msg);
        }
    }
   
最终我们就是通过handleMessage方法来处理的消息，但是这是个空方法，是需要被实现的。

#### 疑问

创建Message的方式：

	1 、Message message = new Messge();
	
	//此时获取的是之前被回收的存在全局消息池中的消息对象
	2、Message message = mHandler.obtain();
	
所以说，如果使用第一种方式创建Message的话，那么是怎么实现Message与Handler的关联的?

#### 综上所述

- 创建一个Handler，并且实现handleMessage()方法

- 如果是在子线程中创建Handler，那么必须先调用Looper.prepare()方法为当前线程创建一个Looper对象。

- Looper对象的构造函数中会持有一个MessageQueue对象，这个是个消息队列，用来存储和取出消息。

- 当使用Handler发送消息的时候，调用MessageQueue的enqueueMessage方法进行消息的入队操作

- 然后消息的出队操作是靠Looper的loop方法来实现的，然后把出队的消息再通过Message所持有的Handler对象的dispatchMessage方法来对消息进行处理


		