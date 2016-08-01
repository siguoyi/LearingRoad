# Volley学习笔记 #
## 什么是Volley ##
Volley是google在Google I/O 2013上发布的一个网络通信框架。Volley是Android平台上的网络通信库，能使网络通信更快，更简单，更健壮。除了简单易用之外，Volley在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作。基本框架如下图所示：

![](http://i.imgur.com/tBCMLQW.png)

## Volley有哪些特点 ##
- 自动调度网络请求
- 多个并发的网络连接
- 通过使用标准的HTTP缓存机制保持磁盘和内存响应的一致
- 支持请求优先级
- 支持取消请求的强大API，可以取消单个请求或多个
- 易于定制
- 健壮性：便于正确的更新UI和获取数据
- 包含调试和追踪工具

## 如何使用Volley ##
在Android Studio里面使用EventBus首先需要添加依赖

    compile 'com.mcxiaoke.volley:library:1.0.19'
依赖添加好了之后，就可以愉快的使用Volley进行网络通信了。主要就是进行了以下三步操作：

1. 创建一个RequestQueue对象。
2. 创建一个*Request对象。
3. 将*Request对象添加到RequestQueue里面。

首先，获取RequestQueue对象，可以调用如下方法获取到：

    RequestQueue mQueue = Volley.newRequestQueue(context);
注意这里拿到的RequestQueue是一个请求队列对象，它可以缓存所有的HTTP请求，然后按照一定的算法并发地发出这些请求。RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的，基本上在每一个需要和网络交互的Activity中创建一个RequestQueue对象就足够了。接下来简单介绍一下几种常用的request的使用方法。
### StringRequest ###
StringRequest是volley框架下最简单的网络请求，我们只需要创建一个StringRequest的实例，创建实例的时候需要传入三个参数:**url**:网络请求的目标url地址；**Listener**:监听response的回调； **ErrorListener**:监听错误response的回调

    StringRequest stringRequest = new StringRequest(url,  
                        new Response.Listener<String>() {  
                            @Override  
                            public void onResponse(String response) {  
                                Log.d(TAG, response);  
                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {  
                                Log.e("TAG", error.getMessage(), error);  
                            }  
                        });  
上面这种请求方式默认为GET请求，如果想要使用POST请求传递一些参数的话，我们需要这样处理

	StringRequest stringRequest = new StringRequest(Method.POST, url, new Response.Listener<String>() {
				            @Override
				            public void onResponse(String response) {
				                Log.d(TAG, response);
				        }, new Response.ErrorListener() {
				            @Override
				            public void onErrorResponse(VolleyError error) {
				                Log.e("TAG", error.getMessage(), error);
				            }
				        }) {
				            @Override
				            protected Map<String, String> getParams() throws AuthFailureError {
				                Map<String, String> map = new HashMap<>();
				                map.put("param1", "value1");
				                map.put("param2", "value2");
				                return map;
				            }
				        }; 
### JsonRequest ###
类似于StringRequest，JsonRequest也是继承自Request类的，不过由于JsonRequest是一个抽象类，因此我们无法直接创建它的实例，那么只能从它的子类入手了。JsonRequest有两个直接的子类，JsonObjectRequest和JsonArrayRequest，从名字上你应该能就看出它们的区别了吧？一个是用于请求一段JSON数据的，一个是用于请求一段JSON数组的。使用JsonObjectRequest需要传入4个参数：**url:**网络请求的目标url地址；**Listener:**监听response的回调； **ErrorListener:**监听错误response的回调

    JsonObjectRequest jsonObjectRequest = new JsonObjectRequest(url, null,
		new Response.Listener<JSONObject>() {
			@Override
			public void onResponse(JSONObject response) {
				Log.d("TAG", response.toString());
			}
		}, new Response.ErrorListener() {
			@Override
			public void onErrorResponse(VolleyError error) {
				Log.e("TAG", error.getMessage(), error);
			}
		});
## 深入理解Volley机制 ##
使用Volley网络框架，首先都需要调用Volley.newRequestQueue(context)获取一个RequestQueue对象

    public static RequestQueue newRequestQueue(Context context) {
	    return newRequestQueue(context, null);
	}
这个方法仅仅只有一行代码，只是调用了newRequestQueue()的方法重载，并给第二个参数传入null。那我们看下带有两个参数的newRequestQueue()方法中的代码，如下所示：

    public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

        String userAgent = "volley/0";
        try {
            String packageName = context.getPackageName();
            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
            userAgent = packageName + "/" + info.versionCode;
        } catch (NameNotFoundException e) {
        }

        if (stack == null) {
            if (Build.VERSION.SDK_INT >= 9) {
                stack = new HurlStack();
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
            }
        }

        Network network = new BasicNetwork(stack);

        RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        queue.start();

        return queue;
    }
从代码里可以看出，首先通过传入的context获取缓存目录，包名，版本号并拼接出一个userAgent。这里判断如果stack是等于null的，则去创建一个HttpStack对象，这里会判断如果手机系统版本号是大于9（Android 2.2）的，则创建一个HurlStack的实例，否则就创建一个HttpClientStack的实例。实际上HurlStack的内部就是使用HttpURLConnection进行网络通讯的，而HttpClientStack的内部则是使用HttpClient进行网络通讯的。

这是因为在Android 2.2版本之前，HttpClient拥有较少的bug，因此使用它是最好的选择。而在Android 2.3版本及以后，HttpURLConnection则是最佳的选择。它的API简单，体积较小，因而非常适用于Android项目。压缩和缓存机制可以有效地减少网络访问的流量，在提升速度和省电方面也起到了较大的作用。

创建好了HttpStack之后，接下来又创建了一个Network对象，它是用于根据传入的HttpStack对象来处理网络请求的，紧接着new出一个RequestQueue对象，并调用它的start()方法进行启动，然后将RequestQueue返回，这样newRequestQueue()的方法就执行结束了。
那么RequestQueue的start()方法内部到底执行了什么东西呢？

    /**
     * Starts the dispatchers in this queue.
     */
    public void start() {
        stop();  // Make sure any currently running dispatchers are stopped.
        // Create the cache dispatcher and start it.
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();

        // Create network dispatchers (and corresponding threads) up to the pool size.
        for (int i = 0; i < mDispatchers.length; i++) {
            NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                    mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
    }

这里先是创建了一个CacheDispatcher的实例，然后调用了它的start()方法，接着在一个for循环里去创建NetworkDispatcher的实例，并分别调用它们的start()方法。这里的CacheDispatcher和NetworkDispatcher都是继承自Thread的，而默认情况下for循环会执行四次，也就是说当调用了Volley.newRequestQueue(context)之后，就会有五个线程一直在后台运行，不断等待网络请求的到来，其中CacheDispatcher是缓存线程，NetworkDispatcher是网络请求线程。得到了RequestQueue之后，我们只需要构建出相应的Request，然后调用RequestQueue的add()方法将Request传入就可以完成网络请求操作了。

    /**
     * Adds a Request to the dispatch queue.
     * @param request The request to service
     * @return The passed-in request
     */
    public <T> Request<T> add(Request<T> request) {
        // Tag the request as belonging to this queue and add it to the set of current requests.
        request.setRequestQueue(this);
        synchronized (mCurrentRequests) {
            mCurrentRequests.add(request);
        }

        // Process requests in the order they are added.
        request.setSequence(getSequenceNumber());
        request.addMarker("add-to-queue");

        // If the request is uncacheable, skip the cache queue and go straight to the network.
        if (!request.shouldCache()) {
            mNetworkQueue.add(request);
            return request;
        }

        // Insert request into stage if there's already a request with the same cache key in flight.
        synchronized (mWaitingRequests) {
            String cacheKey = request.getCacheKey();
            if (mWaitingRequests.containsKey(cacheKey)) {
                // There is already a request in flight. Queue up.
                Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);
                if (stagedRequests == null) {
                    stagedRequests = new LinkedList<Request<?>>();
                }
                stagedRequests.add(request);
                mWaitingRequests.put(cacheKey, stagedRequests);
                if (VolleyLog.DEBUG) {
                    VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
                }
            } else {
                // Insert 'null' queue for this cacheKey, indicating there is now a request in
                // flight.
                mWaitingRequests.put(cacheKey, null);
                mCacheQueue.add(request);
            }
            return request;
        }
    }
可以看到request进来后先判断当前的请求是否可以缓存，如果不能缓存则在直接将这条请求加入网络请求队列，可以缓存的话则在将这条请求加入缓存队列，通过一个Hashmap进行缓存。在默认情况下，每条请求都是可以缓存的，当然我们也可以调用Request的setShouldCache(false)方法来改变这一默认行为。

接下来我们看一下在缓存队列中的request如何进行网络请求，CacheDispatcher为一个缓存线程，所有缓存的request都从这里发起请求

    @Override
    public void run() {
        if (DEBUG) VolleyLog.v("start new dispatcher");
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

        // Make a blocking call to initialize the cache.
        mCache.initialize();

        while (true) {
            try {
                // Get a request from the cache triage queue, blocking until
                // at least one is available.
                final Request<?> request = mCacheQueue.take();
                request.addMarker("cache-queue-take");

                // If the request has been canceled, don't bother dispatching it.
                if (request.isCanceled()) {
                    request.finish("cache-discard-canceled");
                    continue;
                }

                // Attempt to retrieve this item from cache.
                Cache.Entry entry = mCache.get(request.getCacheKey());
                if (entry == null) {
                    request.addMarker("cache-miss");
                    // Cache miss; send off to the network dispatcher.
                    mNetworkQueue.put(request);
                    continue;
                }

                // If it is completely expired, just send it to the network.
                if (entry.isExpired()) {
                    request.addMarker("cache-hit-expired");
                    request.setCacheEntry(entry);
                    mNetworkQueue.put(request);
                    continue;
                }

                // We have a cache hit; parse its data for delivery back to the request.
                request.addMarker("cache-hit");
                Response<?> response = request.parseNetworkResponse(
                        new NetworkResponse(entry.data, entry.responseHeaders));
                request.addMarker("cache-hit-parsed");

                if (!entry.refreshNeeded()) {
                    // Completely unexpired cache hit. Just deliver the response.
                    mDelivery.postResponse(request, response);
                } else {
                    // Soft-expired cache hit. We can deliver the cached response,
                    // but we need to also send the request to the network for
                    // refreshing.
                    request.addMarker("cache-hit-refresh-needed");
                    request.setCacheEntry(entry);

                    // Mark the response as intermediate.
                    response.intermediate = true;

                    // Post the intermediate response back to the user and have
                    // the delivery then forward the request along to the network.
                    mDelivery.postResponse(request, response, new Runnable() {
                        @Override
                        public void run() {
                            try {
                                mNetworkQueue.put(request);
                            } catch (InterruptedException e) {
                                // Not much we can do about this.
                            }
                        }
                    });
                }

            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }
        }
    }
首先，我们能很快发现一个while(true)，表面该缓存线程是一直都在后台运行。该线程会从缓存当中取出响应结果，如果为空的话则把这条请求加入到网络请求队列中，如果不为空的话再判断该缓存是否已过期，如果已经过期了则同样把这条请求加入到网络请求队列中，否则就认为不需
要重发网络请求，直接使用缓存中的数据即可。

在发送请求之后，就是如何获取并解析response了。调用Request的parseNetworkResponse()方法来对数据进行解析，再往后就是将解析出来的数据进行回调了。

    @Override
    public void run() {
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        while (true) {
            long startTimeMs = SystemClock.elapsedRealtime();
            Request<?> request;
            try {
                // Take a request from the queue.
                request = mQueue.take();
            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }

            try {
                request.addMarker("network-queue-take");

                // If the request was cancelled already, do not perform the
                // network request.
                if (request.isCanceled()) {
                    request.finish("network-discard-cancelled");
                    continue;
                }

                addTrafficStatsTag(request);

                // Perform the network request.
                NetworkResponse networkResponse = mNetwork.performRequest(request);
                request.addMarker("network-http-complete");

                // If the server returned 304 AND we delivered a response already,
                // we're done -- don't deliver a second identical response.
                if (networkResponse.notModified && request.hasHadResponseDelivered()) {
                    request.finish("not-modified");
                    continue;
                }

                // Parse the response here on the worker thread.
                Response<?> response = request.parseNetworkResponse(networkResponse);
                request.addMarker("network-parse-complete");

                // Write to cache if applicable.
                // TODO: Only update cache metadata instead of entire record for 304s.
                if (request.shouldCache() && response.cacheEntry != null) {
                    mCache.put(request.getCacheKey(), response.cacheEntry);
                    request.addMarker("network-cache-written");
                }

                // Post the response back.
                request.markDelivered();
                mDelivery.postResponse(request, response);
            } catch (VolleyError volleyError) {
                volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
                parseAndDeliverNetworkError(request, volleyError);
            } catch (Exception e) {
                VolleyLog.e(e, "Unhandled exception %s", e.toString());
                VolleyError volleyError = new VolleyError(e);
                volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
                mDelivery.postError(request, volleyError);
            }
        }
    }
同样的，我们也很容易发现有一个while(true)，所以该线程也是一直在后台运行的。该线程调用Network的performRequest()方法来去发送网络请求，而Network是一个接口，这里具体的实现是BasicNetwork，我们来看下它的performRequest()方法

    @Override
    public NetworkResponse performRequest(Request<?> request) throws VolleyError {
        long requestStart = SystemClock.elapsedRealtime();
        while (true) {
            HttpResponse httpResponse = null;
            byte[] responseContents = null;
            Map<String, String> responseHeaders = Collections.emptyMap();
            try {
                // Gather headers.
                Map<String, String> headers = new HashMap<String, String>();
                addCacheHeaders(headers, request.getCacheEntry());
                httpResponse = mHttpStack.performRequest(request, headers);
                StatusLine statusLine = httpResponse.getStatusLine();
                int statusCode = statusLine.getStatusCode();

                responseHeaders = convertHeaders(httpResponse.getAllHeaders());
                // Handle cache validation.
                if (statusCode == HttpStatus.SC_NOT_MODIFIED) {

                    Entry entry = request.getCacheEntry();
                    if (entry == null) {
                        return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED, null,
                                responseHeaders, true,
                                SystemClock.elapsedRealtime() - requestStart);
                    }

                    // A HTTP 304 response does not have all header fields. We
                    // have to use the header fields from the cache entry plus
                    // the new ones from the response.
                    // http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.5
                    entry.responseHeaders.putAll(responseHeaders);
                    return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED, entry.data,
                            entry.responseHeaders, true,
                            SystemClock.elapsedRealtime() - requestStart);
                }

                // Some responses such as 204s do not have content.  We must check.
                if (httpResponse.getEntity() != null) {
                  responseContents = entityToBytes(httpResponse.getEntity());
                } else {
                  // Add 0 byte response as a way of honestly representing a
                  // no-content request.
                  responseContents = new byte[0];
                }

                // if the request is slow, log it.
                long requestLifetime = SystemClock.elapsedRealtime() - requestStart;
                logSlowRequests(requestLifetime, request, responseContents, statusLine);

                if (statusCode < 200 || statusCode > 299) {
                    throw new IOException();
                }
                return new NetworkResponse(statusCode, responseContents, responseHeaders, false,
                        SystemClock.elapsedRealtime() - requestStart);
            } catch (SocketTimeoutException e) {
                attemptRetryOnException("socket", request, new TimeoutError());
            } catch (ConnectTimeoutException e) {
                attemptRetryOnException("connection", request, new TimeoutError());
            } catch (MalformedURLException e) {
                throw new RuntimeException("Bad URL " + request.getUrl(), e);
            } catch (IOException e) {
                int statusCode = 0;
                NetworkResponse networkResponse = null;
                if (httpResponse != null) {
                    statusCode = httpResponse.getStatusLine().getStatusCode();
                } else {
                    throw new NoConnectionError(e);
                }
                VolleyLog.e("Unexpected response code %d for %s", statusCode, request.getUrl());
                if (responseContents != null) {
                    networkResponse = new NetworkResponse(statusCode, responseContents,
                            responseHeaders, false, SystemClock.elapsedRealtime() - requestStart);
                    if (statusCode == HttpStatus.SC_UNAUTHORIZED ||
                            statusCode == HttpStatus.SC_FORBIDDEN) {
                        attemptRetryOnException("auth",
                                request, new AuthFailureError(networkResponse));
                    } else {
                        // TODO: Only throw ServerError for 5xx status codes.
                        throw new ServerError(networkResponse);
                    }
                } else {
                    throw new NetworkError(networkResponse);
                }
            }
        }
    }
调用了HttpStack的performRequest()方法，这里的HttpStack就是在一开始调用newRequestQueue()方法是创建的实例，默认情况下如果系统版本号大于9就创建的HurlStack对象，否则创建HttpClientStack对象。前面已经说过，这两个对象的内部实际就是分别使用HttpURLConnection和HttpClient来发送网络请求的，我们就不再跟进去阅读了，之后会将服务器返回的数据组装成一个NetworkResponse对象进行返回。

在NetworkDispatcher中收到了NetworkResponse这个返回值后又会调用Request的parseNetworkResponse()方法来解析NetworkResponse中的数据，以及将数据写入到缓存，这个方法的实现是交给Request的子类来完成的，因为不同种类的Request解析的方式也肯定不同。

    public void postResponse(Request<?> request, Response<?> response, Runnable runnable) {
	    request.markDelivered();
	    request.addMarker("post-response");
	    mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, runnable));
	}

其中，在mResponsePoster的execute()方法中传入了一个ResponseDeliveryRunnable对象，就可以保证该对象中的run()方法就是在主线程当中运行的了，我们看下run()方法中的代码是什么样的：

    private class ResponseDeliveryRunnable implements Runnable {
	    private final Request mRequest;
	    private final Response mResponse;
	    private final Runnable mRunnable;
	
	    public ResponseDeliveryRunnable(Request request, Response response, Runnable runnable) {
	        mRequest = request;
	        mResponse = response;
	        mRunnable = runnable;
	    }
	
	    @SuppressWarnings("unchecked")
	    @Override
	    public void run() {
	        // If this request has canceled, finish it and don't deliver.
	        if (mRequest.isCanceled()) {
	            mRequest.finish("canceled-at-delivery");
	            return;
	        }
	        // Deliver a normal response or error, depending.
	        if (mResponse.isSuccess()) {
	            mRequest.deliverResponse(mResponse.result);
	        } else {
	            mRequest.deliverError(mResponse.error);
	        }
	        // If this is an intermediate response, add a marker, otherwise we're done
	        // and the request can be finished.
	        if (mResponse.intermediate) {
	            mRequest.addMarker("intermediate-response");
	        } else {
	            mRequest.finish("done");
	        }
	        // If we have been provided a post-delivery runnable, run it.
	        if (mRunnable != null) {
	            mRunnable.run();
	        }
	   }
	}
调用了Request的deliverResponse()方法，这个就是我们在自定义Request时需要重写的另外一个方法，每一条网络请求的响应都是回调到这个方法中，最后我们再在这个方法中将响应的数据回调到Response.Listener的onResponse()方法中就可以了。
## 总结 ##
总的来说，Volley是一个比较好用的网络请求框架，使用方式也比较简单而且还可以修改一些设置，比如是否缓存以及响应时长等等。合理地在项目中使用Volley，会让我们在网络请求这部分省不少事。
## 参考 ##
[http://blog.csdn.net/guolin_blog/article/details/17656437](http://blog.csdn.net/guolin_blog/article/details/17656437)

[http://www.cnblogs.com/zyw-205520/p/4950357.html](http://www.cnblogs.com/zyw-205520/p/4950357.html)

[http://blog.csdn.net/guolin_blog/article/details/12452307](http://blog.csdn.net/guolin_blog/article/details/12452307)