title: 管理ViewGroup中的触摸事件
date: 2016-02-26 10:09:59
tags:
- Android
- 触摸事件

---

处理**ViewGroup**中的触摸时间值得注意的是，因为ViewGroup本身与带有子View的ViewGroup的触摸事件是不同的。为了确保每个View都能正确的接收到触摸事件，需要覆盖**onInterceptTouchEvent()**方法

### ViewGroup拦截触摸事件

onInterceptTouchEvent()方法在ViewGroup表面或者它的子View的表面检测到触摸事件的时候会被调用。如果onInterceptTouchEvent返回true，**MotionEvent**会被拦截，意味着该事件将不会传递到子view，但是会到达父View的**onTouchEvent()**.

onInterceptTouchEvent方法给父View一个提前看到任何触摸事件的机会，如果你在onInterceptTouchEvent()中返回了true，在子视图处理触摸事件的时候将会接受到一个**ACTION_CANCEL**，然后之后的时间将会直接被发送到父view进行处理。onInterceptTouchEvent()也可以返回false，用来监视在视图层次中触摸事件的传递以及自己的onTouchEvent方法中如何处理触摸事件。

在以下的代码片段中，MyViewGroup类继承ViewGroup，MyViewGroup包含多个子视图，如果你用手指拖过一个水平子视图，子view将不再接收到触摸事件，MyViewGroup将会处理触摸事件通过滚动其内容，如果在子view中点击了按钮，或者垂直滚动子view，父view将不会再拦截他们的触摸事件，因为子view是目标，在这种情况下，onInterceptTouchEvent会返回false，并且MyViewGroup的onTouchEvent()将不会被调用。

	public class MyViewGroup extends ViewGroup{
		private int mTouchSlop;
		
		...
		
		ViewConfiguration vc = ViewConfiguration.get(view.getContext());
		mTouchSlop = vc.getScaledTouchSlop();
		...
		
		@Override
		public boolean onInterceptTouchEvent(MotionEvent ev){
			 /*
         * This method JUST determines whether we want to intercept the motion.
         * If we return true, onTouchEvent will be called and we do the actual
         * scrolling there.
         */
			
			final int action = MotionEventCompat.getActionMasked(ev);
			
			if(action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP){
				mIsScrolling = false;
				return false;//不拦截触摸事件，让子view处理
			}
			
			switch(action){
				case MotionEvent.ACTION_CANCEL:
					if(mIsScrolling){
						//正在滚动，所以拦截触摸事件
						return true;
					}
					
					//如果用户手指水平拖动的距离大于touch slop，开始滚动
					final int xDiff = calculateDistanceX(ex);
					
					//使用ViewConfiguration常量来计算Touch slop
					if(xDiff > mTouchSlop){
						mIsScrolling = true;
						return false;
					}
					break;
			}
			...
		}
		//通常，我们不希望拦截触摸事件，应该被子view处理
		return false;
	}
	
	@Override
	public boolean onTouchEvent(MotionEvent ev){
		//这里是处理触摸事件的地方
		//这个方法只有当触摸事件在onInterceptTouchEvent中被拦截的时候才会被调用。
	}
	}
	
### 使用ViewConfiguration 常量

上面的代码片段使用**ViewConfiguraion**初始化了一个mTouchSlop变量，你可以使用ViewConfiguration类访问Android系统中常用的距离，速度等常量。

Touch Slop通常用于防止当用于执行一些其他的触摸操作，比如触摸屏幕元素造成的意外滚动。

另外两个常用的ViewConfiguration中的方法是**getScaledMinimumFlingVelocity()**和**getScaledMaximumFlingVelocity()**.这两个方法返回fling的说以后的最大最小速度，以像素每秒为单位，比如：

	ViewConfiguration vc = ViewConfiguration.get(view.getContext());
	private int mSlop = vc.getScaledTouchSlop();
	private int mMinFlingVelocity = vc.getScaledMinmumFlingVelocity();
	private int mMaxFlingVelocity = vc.getScaledMaximumFlingVelocity();
	
	...
	
	case MotionEvent.ACTION_MOVE:
		...
		float deltaX = motionEvent.getRawX() - mDownX;
		if(Math.abs(deltaX) > mSlop){
			//发生滑动
		}
		
	...
	
	case MotionEvent.ACTION_UP:
		...
		if(mMinFlingVelocity <= velocityX && velocityX <= mMaxFlingVelocity && velocityY < velocityX){
			
		}
		
### 扩展子视图的可触摸区域

Android提供了一个**TouchDelegate**类来使父view在子view的界限之上扩展子view。这是非常有用的，当子view小的时候，应该有一个大的可接触区域，如果需要的话也可以使用这个方法来缩小子view的接触区域。

在下面的例子中，ImageButton作为delegate view(将被父view扩展触摸区域的view)，下面是布局文件：

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:id="@+id/parent_layout"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		tools:context=".MainActivity">
		
		<ImageButton android:id="@+id/button"
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:background="@null"
			android:src="@drawable/icon"/>
	</RelativeLayout>
	
以下的代码片段如下：

- 获取父view并且再ui线程中发送一个Runnable，确保再在调用**getHitRect()**方法之前父view能够布局好子view，getHitRect()方法获取子view在父view坐标中的矩形区域。
- 找到子view ImageButton，并且调用getHitRect()方法来获取子view的可触摸区域的边界。
- 扩展ImageButton矩形边界的区域。
- 实例化一个TouchDelegate,通过扩大矩形和ImageButton的子view作为参数
- 设置父视图TouchDelegate

如果触摸事件发生再子view的课触摸矩形中，父view将会将触摸事件传给子view并且由子view来处理。

	public class MainActivity extends Activity{
	
		@Override
		protected void onCreate(Bundle savedInstance){
			super.onCreate(savedInstanceState);
			//获取父view
			View parentView = findViewById(R.id.parent_layout);
			
			parentView.post(new Runnable(){
				@Override
				public void run(){
					Rect delegateArea = new Rect();
					ImageButton myButton = (ImageButton)findViewById(R.id.button);
					myButton.setEnabled(true);
					myButton.setOnclickListener(new View.OnClickListener(){
						Toast.makeText(MainActivity.this,"Touch occured within ImageButton touch region.",Toast.LENGTH_SHORT).show();
					});
				}
			});
			
			myButton.getHitRect(delegateArea);
			
			delegateArea.right += 100;
			delegateArea.bottom += 100;
			
			TouchDelegate touchDelegate = new TouchDelegate(delegateArea,myButton);
			
			if(View.class.isInstance(myButton.getParent)){
				((View)myButton.getParent()).setTouchDelegate(touchDelegate);
			}
			
		}
	}