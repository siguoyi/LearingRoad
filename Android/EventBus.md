# EventBus 学习笔记 #
## 什么是EventBus ##
EventBus是[Guava](http://ifeve.com/google-guava/)的事件处理机制，是设计模式中的观察者模式（生产/消费者编程模型）的优雅实现。对于事件监听和发布订阅模式，EventBus是一个非常优雅和简单解决方案，我们不用创建复杂的类和接口层次结构。

观察者模式是比较常用的设计模式之一，虽然有时候在具体代码里，它不一定叫这个名字，比如改头换面叫个Listener，但模式就是这个模式。手工实现一个Observer也不是多复杂的一件事，只是因为这个设计模式实在太常用了，Java就把它放到了JDK里面：Observable和Observer，从JDK 1.0里，它们就一直在那里。从某种程度上说，它简化了Observer模式的开发，至少我们不用再手工维护自己的Observer列表了。不过，如前所述，JDK里的Observer从1.0就在那里了，直到Java 7，它都没有什么改变，就连通知的参数还是Object类型。要知道，Java 5就已经泛型了。Java 5是一次大规模的语法调整，许多程序库从那开始重新设计了API，使其更简洁易用。当然，那些不做应对的程序库，多半也就过时了。这也就是这里要讨论知识更新的原因所在。今天，对于普通的应用，如果要使用Observer模式该如何做呢？答案是Guava的EventBus。
## 怎么使用EventBus ##
在Android Studio里面使用EventBus首先需要添加依赖

    compile 'org.greenrobot:eventbus:3.0.0'
依赖添加好了之后，就可以愉快的使用EventBus了

1.定义EventBus的事件

    public class MessageEvent {
	    public final String message;
	
	    public MessageEvent(String message) {
	        this.message = message;
	    }
	}
2.订阅者需要实现当接收到事件时对应的处理方法，

    public void onEventMainThread(MessageEvent event) {
		Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
		doSomethingWith(event);
    }
3.订阅者同样需要对EventBus进行注册和解注册。只有订阅者注册了，它才能接收到发布的事件。在Android开发里面，Activity和Fragment通常在生命周期里面对EventBus进行注册和解注册。大多情况下，都会在onCreate中进行register，在onDestory中进行unregister 

    @Override
	public void onStart() {
	    super.onStart();
	    EventBus.getDefault().register(this);
	}
	
	@Override
	public void onStop() {
	   EventBus.getDefault().unregister(this);
	    super.onStop();
	}
4.发布EventBus事件：从代码的任意位置发布事件，所有的已注册的能匹配这个事件的订阅者都会接收到这个事件。

    EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
## EventBus的ThreadMode ##
EventBus包含4个ThreadMode：PostThread，MainThread，BackgroundThread，Async

具体的用法，极其简单，方法名为：onEventPostThread， onEventMainThread，onEventBackgroundThread，onEventAsync即可
具体什么区别呢？

- onEventMainThread代表这个方法会在UI线程执行
- onEventPostThread代表这个方法会在当前发布事件的线程执行
- BackgroundThread这个方法，如果在非UI线程发布的事件，则直接执行，和发布在同一个线程中。如果在UI线程发布的事件，则加入后台任务队列，使用线程池一个接一个调用。
- Async 加入后台任务队列，使用线程池调用，注意没有BackgroundThread中的一个接一个。
## EventBus的方法详解 ##
### register ###
    EventBus.getDefault().register(this);
register(this)就是去当前类，遍历所有的方法，找到onEvent开头的然后进行存储。

首先，EventBus.getDefault()其实就是个单例，和我们传统的getInstance一个意思：

     /** Convenience singleton for apps using a process-wide EventBus instance. */
    public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
使用了双重判断的方式，防止并发的问题，还能极大的提高效率。然后register应该是一个普通的方法，我们去看看：

	public void register(Object subscriber) {
		register(subscriber, DEFAULT_METHOD_NAME, false, 0);
	}
	public void register(Object subscriber, int priority) {
		register(subscriber, DEFAULT_METHOD_NAME, false, priority);
	}
	public void registerSticky(Object subscriber) {
		register(subscriber, DEFAULT_METHOD_NAME, true, 0);
	}
	public void registerSticky(Object subscriber, int priority) {
		register(subscriber, DEFAULT_METHOD_NAME, true, priority);
	}
本质上就调用了同一个：

    private synchronized void register(Object subscriber, String methodName, boolean sticky, int priority) {
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass(),
                methodName);
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod, sticky, priority);
        }
    }
这个方法包含4个参数：

- subscriber 是我们扫描类的对象，也就是我们代码中常见的this;
- methodName **这个是写死的**：“onEvent”，用于确定扫描什么开头的方法，可见我们的类中都是以这个开头。
- sticky 暂时不用管
- priority 优先级，优先级越高，在调用的时候会越先调用。

调用内部类SubscriberMethodFinder的**findSubscriberMethods**方法，传入了subscriber 的class，以及methodName，返回一个List<SubscriberMethod>。那么不用说，肯定是去遍历该类内部所有方法，然后根据methodName去匹配，匹配成功的封装成SubscriberMethod，最后返回一个List。

遍历返回的List中的所有方法，把匹配的方法**按照优先级**添加到subscriptionsByEventType（Map，key：eventType ； value：CopyOnWriteArrayList<Subscription> ）中；这个Map其实就是EventBus存储方法的地方。

- eventType是我们方法参数的Class
- Subscription中则保存着subscriber, subscriberMethod（method, threadMode, eventType），priority；包含了执行改方法所需的一切。其中，method是方法名，threadMode是处理事件的线程，eventType是订阅的事件类型。

### post ###
register完毕，知道了EventBus如何存储我们的方法了，下面看看post它又是如何调用我们的方法的。

     /** Posts the given event to the event bus. */
    public void post(Object event) {
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);

        if (postingState.isPosting) {
            return;
        } else {
            postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper(); //判断当前是否是UI线程
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
				//遍历队列中的所有的event，调用postSingleEvent（eventQueue.remove(0), postingState）方法。
                while (!eventQueue.isEmpty()) { 
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
currentPostingThreadState是一个ThreadLocal类型的，里面存储了PostingThreadState；PostingThreadState包含了一个eventQueue和一些标志位。

     private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
        @Override
        protected PostingThreadState initialValue() {
            return new PostingThreadState();
        }
    }
把我们传入的event，保存到了当前线程中的一个变量PostingThreadState的eventQueue中。接下来看postSingleEvent：

    private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<? extends Object> eventClass = event.getClass();
		//根据event的Class，去得到一个List<Class<?>>；其实就是得到event当前对象的Class，
		//以及父类和接口的Class类型；主要用于匹配，比如你传入Dog extends Animal，他会把Animal也装到该List中。
        List<Class<?>> eventTypes = findEventTypes(eventClass);
        boolean subscriptionFound = false;
        int countTypes = eventTypes.size();
		//遍历所有的Class，到subscriptionsByEventType去查找subscriptions
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            CopyOnWriteArrayList<Subscription> subscriptions;
            synchronized (this) {
                subscriptions = subscriptionsByEventType.get(clazz);
            }
            if (subscriptions != null && !subscriptions.isEmpty()) {
                for (Subscription subscription : subscriptions) {
                    postingState.event = event;
                    postingState.subscription = subscription;
                    boolean aborted = false;
                    try {
                        postToSubscription(subscription, event, postingState.isMainThread);
                        aborted = postingState.canceled;
                    } finally {
                        postingState.event = null;
                        postingState.subscription = null;
                        postingState.canceled = false;
                    }
                    if (aborted) {
                        break;
                    }
                }
                subscriptionFound = true;
            }
        }
        if (!subscriptionFound) {
            Log.d(TAG, "No subscribers registered for event " + eventClass);
            if (eventClass != NoSubscriberEvent.class && eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }
将我们的event，即post传入的实参；以及postingState传入到postSingleEvent中。

遍历每个subscription,依次去调用postToSubscription(subscription, event, postingState.isMainThread);这个方法就是去反射执行方法了，大家还记得在register，if(sticky)时，也会去执行这个方法。下面看它如何反射执行：

    private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
        case PostThread:
            invokeSubscriber(subscription, event);
            break;
        case MainThread:
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case BackgroundThread:
            if (isMainThread) {
                backgroundPoster.enqueue(subscription, event);
            } else {
                invokeSubscriber(subscription, event);
            }
            break;
        case Async:
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }
subscription包含了所有执行需要的东西，大致有:subscriber, subscriberMethod（method, threadMode, eventType）, priority；根据**subscription.threadMode**去判断应该在哪个线程去执行该方法。

- **case PostThread：**直接反射调用；也就是说在当前的线程直接调用该方法；
- **case MainThread：**首先去判断当前如果是UI线程，则直接调用；否则： mainThreadPoster.enqueue(subscription, event);把当前的方法加入到队列，然后直接通过handler去发送一个消息，在handler的handleMessage中，去执行我们的方法。说白了就是通过Handler去发送消息，然后执行的。
- **case BackgroundThread：**如果当前非UI线程，则直接调用；如果是UI线程，则将任务加入到后台的一个队列，最终由Eventbus中的一个线程池去调用executorService = Executors.newCachedThreadPool();。
- **case Async：**将任务加入到后台的一个队列，最终由Eventbus中的一个线程池去调用；线程池与BackgroundThread用的是同一个。
- 这么说BackgroundThread和Async有什么区别呢？BackgroundThread中的任务，一个接着一个去调用，中间使用了一个布尔型变量handlerActive进行的控制。Async则会动态控制并发。
### 其余方法 ###
介绍了register和post；大家获取还能想到一个词sticky，在register中，如何sticky为true，会去stickyEvents去查找事件，然后立即去post；那么这个stickyEvents何时进行保存事件呢？其实evevntbus中，除了post发布事件，还有一个方法也可以：

     public void postSticky(Object event) {
        synchronized (stickyEvents) {
            stickyEvents.put(event.getClass(), event);
        }
        // Should be posted after it is putted, in case the subscriber wants to remove immediately
        post(event);
    }
和post功能类似，但是会把方法存储到stickyEvents中去；
## 小结 ##
简单的对EventBus做一个小结：register会把当前类中匹配的方法，存入一个map，而post会根据实参去map查找进行反射调用，即在一个单例内部维持着一个map对象存储了一堆的方法；post无非就是根据参数去查找方法，进行反射调用。
## 参考 ##
[http://blog.csdn.net/lmj623565791/article/details/40794879](http://blog.csdn.net/lmj623565791/article/details/40794879)

[http://blog.csdn.net/lmj623565791/article/details/40920453](http://blog.csdn.net/lmj623565791/article/details/40920453)

[https://github.com/greenrobot/EventBus](https://github.com/greenrobot/EventBus)

[http://www.cnblogs.com/peida/p/EventBus.html](http://www.cnblogs.com/peida/p/EventBus.html)