title: ImageLoader分析
date: 2015-11-30 10:48:28
tags:
- ImageLoader
- Android

---
### ImageLoader

用于加载和显示图片到Imageview上，是个单例；

#### 包含的类

- ImageLoaderConfiguration	ImageLoader的一些配置信息，包括图片宽高，内存大小等等。
- ImageLoaderEngine 用来执行加载和显示图片的具体类
- ImageLoaderListener 加载图片过程中的一些监听方法

#### displayImage()的一些重载方法

将添加显示图片的任务到执行池中，图片将直接被显示在ImageView上

所包含的参数：

- String uri 图片地址
- ImageAware imageAware  用来显示图片的地方，相当于ImageView
- DisplayImageOptions options 显示图片的一些参数
- ImageLoadingListener listener 加载图片的一些监听方法，比如说失败，成功，取消等等，在UI线程中执行
- ImageLoadingProgressListener progressListener 图片加载的进度，在UI线程中执行

具体代码:

	public void displayImage(String uri, ImageAware imageAware, DisplayImageOptions options,
				ImageLoadingListener listener, ImageLoadingProgressListener progressListener) {
			checkConfiguration();
			if (imageAware == null) {
				throw new IllegalArgumentException(ERROR_WRONG_ARGUMENTS);
			}
			if (listener == null) {
				listener = defaultListener;
			}
			if (options == null) {
				options = configuration.defaultDisplayImageOptions;
			}

			if (TextUtils.isEmpty(uri)) {
				engine.cancelDisplayTaskFor(imageAware);
				listener.onLoadingStarted(uri, imageAware.getWrappedView());
				if (options.shouldShowImageForEmptyUri()) {
					imageAware.setImageDrawable(options.getImageForEmptyUri(configuration.resources));
				} else {
					imageAware.setImageDrawable(null);
				}
				listener.onLoadingComplete(uri, imageAware.getWrappedView(), null);
				return;
			}

			ImageSize targetSize = ImageSizeUtils.defineTargetSizeForView(imageAware, configuration.getMaxImageSize());
			String memoryCacheKey = MemoryCacheUtils.generateKey(uri, targetSize);
			engine.prepareDisplayTaskFor(imageAware, memoryCacheKey);

			listener.onLoadingStarted(uri, imageAware.getWrappedView());

		//从内存中根据所存储的key来获取图片
		//key的存储规则:[imageUri]_[width]x[height]
			Bitmap bmp = configuration.memoryCache.get(memoryCacheKey);
			//如果图片存在并且没有被内存回收的情况下
			if (bmp != null && !bmp.isRecycled()) {
				L.d(LOG_LOAD_IMAGE_FROM_MEMORY_CACHE, memoryCacheKey);

				if (options.shouldPostProcess()) {
				//如果允许对图片进行处理
					ImageLoadingInfo imageLoadingInfo = new ImageLoadingInfo(uri, imageAware, targetSize, memoryCacheKey,
							options, listener, progressListener, engine.getLockForUri(uri));
					ProcessAndDisplayImageTask displayTask = new ProcessAndDisplayImageTask(engine, bmp, imageLoadingInfo,
							defineHandler(options));
					if (options.isSyncLoading()) {
						displayTask.run();
					} else {
						engine.submit(displayTask);
					}
				} else {
					options.getDisplayer().display(bmp, imageAware, LoadedFrom.MEMORY_CACHE);
					listener.onLoadingComplete(uri, imageAware.getWrappedView(), bmp);
				}
			} else {
				if (options.shouldShowImageOnLoading()) {
					//如果设置了加载中所需要显示的图片
					imageAware.setImageDrawable(options.getImageOnLoading(configuration.resources));
				} else if (options.isResetViewBeforeLoading()) {
					imageAware.setImageDrawable(null);
				}

				ImageLoadingInfo imageLoadingInfo = new ImageLoadingInfo(uri, imageAware, targetSize, memoryCacheKey,
						options, listener, progressListener, engine.getLockForUri(uri));
				LoadAndDisplayImageTask displayTask = new LoadAndDisplayImageTask(engine, imageLoadingInfo,
						defineHandler(options));
				if (options.isSyncLoading()) {
					//是否同步加载，默认为false
					displayTask.run();
				} else {
					engine.submit(displayTask);
				}
			}
		}
		
#### loadImage()的一些重载方法

添加加载图片任务到执行池中，图片将在ImageLoadingListener的**OnLoadingComplete**方法中被返回回来

具体参数：和displayImage方法的参数相同

- String uri
- ImageSize targetImageSize
- DisplayImageOptions options
- ImageLoadingListener listner
- ImageLoadingProgressListener progressListener

具体代码：

	public void loadImage(String uri, ImageSize targetImageSize, DisplayImageOptions options,
				ImageLoadingListener listener, ImageLoadingProgressListener progressListener) {
			checkConfiguration();
			if (targetImageSize == null) {
				targetImageSize = configuration.getMaxImageSize();
			}
			if (options == null) {
				options = configuration.defaultDisplayImageOptions;
			}

			NonViewAware imageAware = new NonViewAware(uri, targetImageSize, ViewScaleType.CROP);
			displayImage(uri, imageAware, options, listener, progressListener);
		}
		
#### loadImageSync()

同步加载图片

具体参数：

- String uri
- DisplayImageOptions options 默认显示图片参数

具体代码

	public Bitmap loadImageSync(String uri, ImageSize targetImageSize, DisplayImageOptions options) {
			if (options == null) {
				options = configuration.defaultDisplayImageOptions;
			}
			options = new DisplayImageOptions.Builder().cloneFrom(options).syncLoading(true).build();

			SyncImageLoadingListener listener = new SyncImageLoadingListener();
			loadImage(uri, targetImageSize, options, listener);
			return listener.getLoadedBitmap();
		}
		
#### 其他方法

- pause() 暂停ImageLoader，所有新添加的加载和显示图片的任务都会被暂停，然而正在进行中的任务将不会被暂停
- resume() 重新恢复暂停中的任务
- stop() 取消所有的进行和等待中的任务
- destroy() 停止imageLoader并且清空当前的配置信息

###　ImageLoaderConfiguration

ImageLoader的配置信息

主要元素：

	final Resources resources;

	final int maxImageWidthForMemoryCache;
	final int maxImageHeightForMemoryCache;
	final int maxImageWidthForDiskCache;
	final int maxImageHeightForDiskCache;
	final BitmapProcessor processorForDiskCache;

	final Executor taskExecutor;
	final Executor taskExecutorForCachedImages;
	final boolean customExecutor;
	final boolean customExecutorForCachedImages;

	final int threadPoolSize;
	final int threadPriority;
	final QueueProcessingType tasksProcessingType;

	final MemoryCache memoryCache;
	final DiskCache diskCache;
	final ImageDownloader downloader;
	final ImageDecoder decoder;
	final DisplayImageOptions defaultDisplayImageOptions;

	final ImageDownloader networkDeniedDownloader;
	final ImageDownloader slowNetworkDownloader;
	
	memoryCacheSize:默认为app可用内存的1/8
	
### ImageLoaderEngine

用来执行显示图片的任务

重要方法：

	//提交任务到执行池中
	void submit(final LoadAndDisplayImageTask task) {
			taskDistributor.execute(new Runnable() {
				@Override
				public void run() {
				//从disk缓存中获取图片
					File image = configuration.diskCache.get(task.getLoadingUri());
					boolean isImageCachedOnDisk = image != null && image.exists();
					initExecutorsIfNeed();
					if (isImageCachedOnDisk) {
						//如果disk中存在改图片
						taskExecutorForCachedImages.execute(task);
					} else {
						//如果不存在
						taskExecutor.execute(task);
					}
				}
			});
		}
		
### LoadAndDisplayTask

用来加载和显示图片的具体的任务

相关属性和方法:

- ImageDownloader:用来下载图片的接口
- ImageDecoder:解码返回的Bitmap图片

主要方法:

	@Override
		public void run() {
			if (waitIfPaused()) return;
			if (delayIfNeed()) return;

			ReentrantLock loadFromUriLock = imageLoadingInfo.loadFromUriLock;
			L.d(LOG_START_DISPLAY_IMAGE_TASK, memoryCacheKey);
			if (loadFromUriLock.isLocked()) {
				L.d(LOG_WAITING_FOR_IMAGE_LOADED, memoryCacheKey);
			}

			loadFromUriLock.lock();
			Bitmap bmp;
			try {
				checkTaskNotActual();
			//先从内存中加载图片
				bmp = configuration.memoryCache.get(memoryCacheKey);
				if (bmp == null || bmp.isRecycled()) {
				//如果图片不存在
				//从Disk和网络加载图片
					bmp = tryLoadBitmap();
					if (bmp == null) return; // listener callback already was fired

					checkTaskNotActual();
					checkTaskInterrupted();

					if (options.shouldPreProcess()) {
						L.d(LOG_PREPROCESS_IMAGE, memoryCacheKey);
						bmp = options.getPreProcessor().process(bmp);
						if (bmp == null) {
							L.e(ERROR_PRE_PROCESSOR_NULL, memoryCacheKey);
						}
					}

					if (bmp != null && options.isCacheInMemory()) {
						L.d(LOG_CACHE_IMAGE_IN_MEMORY, memoryCacheKey);
						configuration.memoryCache.put(memoryCacheKey, bmp);
					}
				} else {
				//从内存中加载图片
					loadedFrom = LoadedFrom.MEMORY_CACHE;
					L.d(LOG_GET_IMAGE_FROM_MEMORY_CACHE_AFTER_WAITING, memoryCacheKey);
				}

				if (bmp != null && options.shouldPostProcess()) {
					L.d(LOG_POSTPROCESS_IMAGE, memoryCacheKey);
					bmp = options.getPostProcessor().process(bmp);
					if (bmp == null) {
						L.e(ERROR_POST_PROCESSOR_NULL, memoryCacheKey);
					}
				}
				checkTaskNotActual();
				checkTaskInterrupted();
			} catch (TaskCancelledException e) {
				fireCancelEvent();
				return;
			} finally {
				loadFromUriLock.unlock();
			}

			DisplayBitmapTask displayBitmapTask = new DisplayBitmapTask(bmp, imageLoadingInfo, engine, loadedFrom);
			runTask(displayBitmapTask, syncLoading, handler, engine);
		}
		
		//tryLoadBitmap()方法用于从Disk和Network加载图片
		
	private Bitmap tryLoadBitmap() throws TaskCancelledException {
		Bitmap bitmap = null;
		try {
			File imageFile = configuration.diskCache.get(uri);
			if (imageFile != null && imageFile.exists() && imageFile.length() > 0) {
				L.d(LOG_LOAD_IMAGE_FROM_DISK_CACHE, memoryCacheKey);
				loadedFrom = LoadedFrom.DISC_CACHE;

				checkTaskNotActual();
				bitmap = decodeImage(Scheme.FILE.wrap(imageFile.getAbsolutePath()));
			}
			if (bitmap == null || bitmap.getWidth() <= 0 || bitmap.getHeight() <= 0) {
				L.d(LOG_LOAD_IMAGE_FROM_NETWORK, memoryCacheKey);
				loadedFrom = LoadedFrom.NETWORK;

				String imageUriForDecoding = uri;
				if (options.isCacheOnDisk() && tryCacheImageOnDisk()) {
					imageFile = configuration.diskCache.get(uri);
					if (imageFile != null) {
						imageUriForDecoding = Scheme.FILE.wrap(imageFile.getAbsolutePath());
					}
				}

				checkTaskNotActual();
				bitmap = decodeImage(imageUriForDecoding);

				if (bitmap == null || bitmap.getWidth() <= 0 || bitmap.getHeight() <= 0) {
					fireFailEvent(FailType.DECODING_ERROR, null);
				}
			}
		} catch (IllegalStateException e) {
			fireFailEvent(FailType.NETWORK_DENIED, null);
		} catch (TaskCancelledException e) {
			throw e;
		} catch (IOException e) {
			L.e(e);
			fireFailEvent(FailType.IO_ERROR, e);
		} catch (OutOfMemoryError e) {
			L.e(e);
			fireFailEvent(FailType.OUT_OF_MEMORY, e);
		} catch (Throwable e) {
			L.e(e);
			fireFailEvent(FailType.UNKNOWN, e);
		}
		return bitmap;
	}

### ImageDownloader

用于下载图片

### BaseImageDownloader

	//createConnection()创建连接
	
	protected HttpURLConnection createConnection(String url,Object extra) throws IOException{
		String encodedUrl = Uri.encode(url,ALLOWED_URI_CHARS);
		HttpURLConnection conn = (HttpURLConnection)new URL(encodedUrl).openConnection();
		conn.setConnectTimeout(connectTimeout);
		conn.setReadTimeout(readTimeout);
		return conn;
	}
	
	//从网络获取流
	protected InputStream getStreamFromNetwork(String imageUri,Object extra) throws IOException{
		HttpURLConnection conn = createConnection(imageUri,extra);
		int redirectCount = 0;
		while(conn.getResponseCode()/100==3&&redirectCount<MAX_REDIRECT_COUNT){
			//如果响应码等于3并且重定向的次数小于5
			//那么就用重定向的地址重新创建连接
			conn=createConnection(conn.getHeaderField("Location"),extra);
			redirect++;
		}
		InputStream imageStream;
		try {
			imageStream = conn.getInputStream();
		} catch (IOException e) {
			// Read all data to allow reuse connection (http://bit.ly/1ad35PY)
			IoUtils.readAndCloseStream(conn.getErrorStream());
			throw e;
		}
		//如果响应码不等于200，那么网络请求失败
		if (!shouldBeProcessed(conn)) {
			IoUtils.closeSilently(imageStream);
			throw new IOException("Image request failed with response code " + conn.getResponseCode());
		}

		return new ContentLengthInputStream(new BufferedInputStream(imageStream, BUFFER_SIZE), conn.getContentLength());
	}
	
	//还有其他的一些获取流的方法
	
### MemoryCache 

内存缓存

实现类：

- LruCacheMemory
- LimitedAgeMemoryCache
- BaseMemoryCache

	- LimitedMemoryCache
	- WeakMemoryCache
	
### LruCacheMemory

将缓存的图片保存为强引用，当一个位图被访问时，就会被移动到队列的头部，当一个图片被添加进缓存的时候，会被放到队列的末尾。便于垃圾回收

### LimitedAgeMemoryCache

用来缓存图片，如果缓存时间超过规定值，那么将会被移除

### DiskCache

磁盘缓存

- BaseDiskCache 基本的磁盘缓存类
- LimitedAgeDiskCache 如果加载后的时间超过定义的时间，将会被删除，缓存大小未被定义
- UnlimitedDiskCache