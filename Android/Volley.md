# Volley学习笔记 #
## 什么是Volley ##
Volley是google在Google I/O 2013上发布的一个网络通信框架。Volley是Android平台上的网络通信库，能使网络通信更快，更简单，更健壮。除了简单易用之外，Volley在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作
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
