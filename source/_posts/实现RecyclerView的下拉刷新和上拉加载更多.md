title: 实现RecyclerView的下拉刷新和上拉加载更多
date: 2015-11-04 16:28:14
tags:
- Android
- RecyclerView

---

这两天项目中牵涉了对于RecyclerView的上拉加载更多和下拉刷新的功能，所以总结一下:

我们的下拉刷新使用的是**android.support.v4.widget.SwipeRefreshLayout**,而上拉加载更多使用的方案是通过给RecyclerView添加**footerview**来实现的。

所以代码是这样的:

	mSwipeRefreshLayout.setOnRefreshListener(this);

	@Override
	public void onRefresh(){
		//在这里执行下拉刷新的功能
	}

	//以下是上拉加载更多的方法
	int lastVisibleItem//最后一条可见的item
	mRvMessage.addOnScrollListener(new RecyclerView.OnScrollListener() {
	            @Override
	            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
	                super.onScrollStateChanged(recyclerView, newState);
	                int totalCount = mLinearLayoutManager.getItemCount();//RecyclerView中的总的条目的数量(此处代表的是可见的和不可见的总数)
	                if(newState == RecyclerView.SCROLL_STATE_IDLE && lastVisibleItem + 1==totalCount){
	                		//此处添加加载更多的代码，一般为取数据库或者加载网络等
	                }
	            }

	            @Override
	            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
	                super.onScrolled(recyclerView, dx, dy);
	                //每次滑动的时候要更新最后一条可见的item的id
	                lastVisibleItem = mLinearLayoutManager.findLastVisibleItemPosition();
	            }
	        });