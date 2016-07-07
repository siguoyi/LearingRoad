# RxJava: Reactive Extensions for the JVM #
最近项目中有用到RxJava来进行响应式编程，目前对这部分内容还比较陌生，于是决定开始一波RxJava相关内容的学习，想尽快掌握这门技术。在网上还是有很多相关的资源，我的博客仅作为相关内容的搬运工，希望在搬运的过程中提高自己的能力。在此推荐一个[RxJava的专题网站](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0430/2815.html)，并在博文开始之前感谢**[“泡在网上的日子”](http://www.jcodecraeer.com/)**对RxJava相关知识较为详尽的整理。
## 什么是RxJava ##
RxJava is a Java VM implementation of [Reactive Extensions](http://reactivex.io/): a library for composing asynchronous and event-based programs by using observable sequences.

**RxJava是在Java虚拟机上的响应式拓展：一个在JVM上使用可观测的序列来组成异步的、基于事件的程序的库**

It extends the [observer pattern](https://en.wikipedia.org/wiki/Observer_pattern) to support sequences of data/events and adds operators that allow you to compose sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization, thread-safety and concurrent data structures.

**它将观察者模式拓展为支持数据/事件序列并增加了操作，这些操作允许你陈述一样将序列组合到一起并且将关注点从事件中抽离出来，比如低级别的线程，同步，线程安全和并发数据结构。**

- Zero Dependencies
- **零依赖**
- < 1MB Jar
- **Jar包小于1MB**
- Java 6+ & Android 2.3+
- **Java 6 以上，Android 2.3 以上支持**
- Java 8 lambda support
- **支持Java 8 的lambda**
- Polyglot (Scala, Groovy, Clojure and Kotlin)
- **多语言支持（Scala, Groovy, Clojure and Kotlin）**
- Async or synchronous execution
- **异步或同步执行**
- Virtual time and schedulers for parameterized concurrency
- **参数化的并发虚拟时间和调度**
## 如何导入RxJava ##
### Gradle ###
    compile 'io.reactivex:rxjava:1.0.14'
    compile 'io.reactivex:rxandroid:1.0.1' //for Android 
### Maven ###
    <dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>1.0.14</version>
	</dependency>
## RxJava基础 ##
### 基础 ###

----------

RxJava 的异步实现，是通过一种扩展的观察者模式来实现的。观察者模式面向的需求是：A 对象（观察者）对 B 对象（被观察者）的某种变化高度敏感，需要在 B 变化的一瞬间做出反应。举个例子，在Android开发中设置的OnClickListener就是使用了观察者模式，当点击事件发生时会自动通知（或回调）相应的onClick方法执行相应的操作。

RxJava 有四个基本概念：Observable (可观察者，即被观察者)、 Observer/Subscriber (观察者)、 subscribe (订阅)、事件。Observable 和Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

RxJava最核心的两个东西是**Observables（被观察者，事件源）**和 **Subscribers，Observers（观察者）**。Observables发出一系列事件，Subscribers处理这些事件。这里的事件可以是任何你感兴趣的东西 （触摸事件，web接口调用返回的数据。。。）

一个Observable可以发出零个或者多个事件，直到结束或者出错。每发出一个事件，都会有onNext()，onComplete()，onError()中的方法相应。

- **onNext()**：响应普通事件队列
- **onComplete()**：事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的onNext() 发出时，需要触发 onCompleted() 方法作为标志。
- **onError()**：事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
 
Rxjava的看起来很想设计模式中的观察者模式，但是有一点明显不同，那就是**如果一个Observerble没有任何的的Subscriber，那么这个Observable是不会发出任何事件的。**
### 如何实现RxJava ###

----------

#### 创建观察者Observer ####
**Observer 即观察者，它决定事件触发的时候将有怎样的行为。**

RxJava 中的 **Observer 接口**的实现方式：

    Observer<String> observer = new Observer<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Next: " + s);
    }
 
    @Override
    public void onCompleted() {
        Log.d(tag, "Completed!");
    }
 
    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error!");
    }};
RxJava 中的 **Subscriber 接口**的实现方式：

    Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) {
        Log.d(tag, "Item: " + s);
    }
 
    @Override
    public void onCompleted() {
        Log.d(tag, "Completed!");
    }
 
    @Override
    public void onError(Throwable e) {
        Log.d(tag, "Error!");
    }};

#### 创建被观察者Observable ####
**Observable 即被观察者，它决定什么时候触发事件以及触发怎样的事件。创建被观察者有一下三种方法：create(...)，just(...)，from(...)**

**1. create(...)** RxJava 使用 create() 方法来创建一个 Observable ，并为它定义事件触发规则

    Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("Hello");
        subscriber.onNext("Rx");
        subscriber.onNext("Java");
        subscriber.onCompleted();
    }});

**2. just(...)** just(T...): 将传入的参数依次发送出来

    Observable observable = Observable.just("Hello", "Rx", "Java");
	// 将会依次调用：
	// onNext("Hello");
	// onNext("Rx");
	// onNext("Java");
	// onCompleted();
**3. from(...)** from(T[]) / from(Iterable<? extends T>) : 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来。

    String[] words = {"Hello", "Rx", "Java"};
	Observable observable = Observable.from(words);
	// 将会依次调用：
	// onNext("Hello");
	// onNext("Rx");
	// onNext("Java");
	// onCompleted();
上面 just(T...) 的例子和 from(T[]) 的例子，都和之前的 create(OnSubscribe) 的例子是等价的。

#### 订阅suscribe，建立观察者与被观察者之间的联系 ####
创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：

    observable.subscribe(observer);
	// 或者：
	observable.subscribe(subscriber);

## RxJava的操作符 ##
创建和订阅一个 Observable 是足够简单的，可能这并不是非常有用的，但这只是用 RxJava 的一个开始。通过调用操作符，任何的 Observable 都能进行输出转变，多个Operators 能链接到 Observable上。RxJava 提供了对事件序列进行变换的支持，这是它的核心功能之一，**所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。**
### map() ###
map()是RXJava操作符中最简单的一个，它所实现的功能为对事件对象的直接转换，并且是**一对一的转换。**如下面所示的例子，我通过map()操作符进行了如下操作：通过new Date()方法创建一个当前时间的对象，通过map()操作符对这个对象进行格式的转换后输出为字符串，并在字符串前面加上前缀“Current time: ”，最终将字符串显示到TextView控件当中。

    Observable.just(new Date()).map(new Func1<Date, String>() {
            @Override
            public String call(Date date) {
                String s = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(date);
                return "Current time: " + s;
            }
        }).subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                sb.append(s).append("\n");
                tv_rx_text.setText(sb.toString());
            }
        });
### flatMap() ###
有了map()操作符之后，我们已经能够比较容易的对事件对象进行所需的转换操作，但是您也能发现在上一部分我强调了一下map()操作符所进行的转换是一对一的，这就是这个操作符的缺陷。如果我们需要对一个事件中的对象进行转换，而这个事件包含N个事件对象，那么我们相对于要进行N次的map()操作，这无疑是很浪费的。有人说了我们可以将事件中的N个对象封转到一个List里面，我们在转换的时候对List进行遍历，分别对每个事件对象进行转换。这能实现现在的需求吗？答案是肯定的，但是如果我们不想去封转一个List呢，这个时候一个新的操作符flatMap()就出现了，这个操作符实现的功能也是对事件对象的转换，并且它是支持**一对多的转换。**

    Subscriber<Student.Course> subscriber = new Subscriber<Student.Course>() {
            @Override
            public void onCompleted() {
                sb.append("onCompleted\n");
                tv_rx_text.setText(sb.toString());
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(Student.Course course) {
                sb.append(course.getCourse() + "\n");
                tv_rx_text.setText(sb.toString());
            }
        };

        Observable.just(student[i]).flatMap(new Func1<Student, Observable<Student.Course>>() {
            @Override
            public Observable<Student.Course> call(Student student) {
                return Observable.from(student.getCourseList());
            }
        }).subscribe(subscriber);
如上述代码所示，这个示例中我们做了这么一件事情：我们传入了一个Student对象（里面包含一个内部类Course用来保存student的课程信息）,我们通过flatMap()对传入的Student对象进行转换，将Student对象转换成了一个Course对象，并用这个Course对象构造一个Observable对象返回进行处理，最后将Course对象中包含的课程信息显示在TextView中。

从上面的代码中可以看出，map()和flatMap()有一个共同点：都是把传入的参数转换之后返回另一个对象。但是和map()方法不同的是，flatMap()中返回的是一个Observable对象，并且这个Observable不是直接被发送到了Subscriber的回调方法中。flatMap()的原理是这样的：

1. 使用传入的事件对象创建一个 Observable 对象；
2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；
3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable ，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。

这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 flatMap() 所谓的 flat。
### filter() ###
filter()操作符是可以对Observable流程的数据进行一层过滤处理，filter() 返回为 false 的值将不会发出到 Subscriber。

	int num = (int)(Math.random()*100);    
	Observable.just(num).filter(new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                boolean isEvenNumber = integer % 2 == 0;
                return isEvenNumber;
            }
        }).subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                sb.append("Current number: " + integer + "\n");
                tv_rx_text.setText(sb.toString());
            }
        });
上述示例为随机生成一个0~100之间的数字，当数字为偶数时，对数字按“Current number: ”的格式显示在TextView中。
### interval() ###
对于轮询器大家一定不陌生，开发中无论是Java的Timer+TimeTask , 还是Android的Hanlder都可实现。当然在RxJava中也有这样的实现方式，那就是使用interval()操作符。我们使用interval()实现一个10s的计时器，每间隔一面在TextView中更新一下时间，直到10s。代码如下：

	Observable.interval(1, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Long>() {
                    @Override
                    public void onCompleted() {
                        sb.append("Complete ! \n");
                        tvRxText.setText(sb.toString());
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Long aLong) {
                        if(aLong < 10){
                            sb.append(aLong + 1 + " s" + "\n");
                            tvRxText.setText(sb.toString());
                        }else {
                            onCompleted();
                        }
                    }
                });
interval()方法的第一个参数为每次更新的时间间隔，第二个参数为该时间间隔的单位。通过运行此代码，我们发现确实能实现10s计时器的功能，但是到了10s以后，计时器仍未停止，它会一直下去（TextView中的“Complete !”依然每隔一秒打印一次）。所以这其实也是一种浪费，网上有说可以在onNext()里计算时间，达到要求时进行解绑（**目前我还没找到解绑interval的方法，如果您知道，请赐教**）。在这种情况下，take()操作符应运而生，它和interval()能完美结合实现计时器的功能，接下来我们来看一下take()操作符的使用。
### take() ###
take从字面意思上可以理解就是“拿，取”的意思。所以take()所起的作用也就是取的作用，根据传入参数的数值N来获取前N个onNext()的结果，达到指定数值之后，调用onCompleted()完成此次计时操作。

    Observable.interval(1, TimeUnit.SECONDS)
                .take(10)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Long>() {
                    @Override
                    public void onCompleted() {
                        sb.append("Complete ! \n");
                        tvRxText.setText(sb.toString());
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Long aLong) {
                        sb.append(aLong + 1 + " s" + "\n");
                        tvRxText.setText(sb.toString());
                    }
                });
take()操作符的使用方法如上所示，虽然比较简单，但是却很好的解决了interval计时不能停的问题。

总的来说，RxJava中的操作符可以说是RxJava中比较核心的部分，合理的运用这些操作符会让我们的工作事半功倍。

## 线程调度 ##
在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，如果只用上面的方法，实现出来的只是一个同步的观察者模式。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。而要实现异步，则需要用到 RxJava 的另一个概念： **Scheduler**

在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。

### Scheduler 的 API ###
在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：

- Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
- Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
- Schedulers.io(): I/O 操作**（读写文件、读写数据库、网络信息交互等）**所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个**无数量上限的线程池**，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。 **不要把计算工作放在 io() 中，可以避免创建不必要的线程。**
- Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 **CPU 密集型计算**，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。**不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。**
- Schedulers.from(executor):使用指定的Executor作为调度器
- Schedulers.trampoline():当其它排队的任务完成后，在当前线程排队开始执行
- 另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。

1. subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。
2. observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

	    Observable.just(1, 2, 3, 4)
	    .subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
	    .observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
	    .subscribe(new Action1<Integer>() {
	        @Override
	        public void call(Integer number) {
	            Log.d(tag, "number:" + number);
	        }
	    });
上面这段代码中，由于 subscribeOn(Schedulers.io()) 的指定，被创建的事件的内容 1、2、3、4 将会在 IO 线程发出；而由于observeOn(AndroidScheculers.mainThread()) 的指定，因此 subscriber 数字的打印将发生在主线程 。事实上，这种在subscribe() 之前写上两句 subscribeOn(Scheduler.io()) 和 observeOn(AndroidSchedulers.mainThread()) 的使用方式非常常见，它适用于多数的 『后台线程取数据，主线程显示』的程序策略。

**注意：多次使用subscribeOn()和observeOn()可以切换事件的发生线程和回调线程。**
### Scheduler拓展 ###
除了将这些调度器传递给RxJava的Observable操作符，你也可以用它们调度你自己的任务。下面的示例展示了Scheduler.Worker的用法：

    worker = Schedulers.newThread().createWorker();
	worker.schedule(new Action0() {

	    @Override
	    public void call() {
	        yourWork();
	    }

	});
	// some time later...
	worker.unsubscribe();
#### 递归调度器 ####
要调度递归的方法调用，你可以使用schedule，然后再用schedule(this)，示例：

    worker = Schedulers.newThread().createWorker();
	worker.schedule(new Action0() {
	
	    @Override
	    public void call() {
	        yourWork();
	        // recurse until unsubscribed (schedule will do nothing if unsubscribed)
	        worker.schedule(this);
	    }
	
	});
	// some time later...
	worker.unsubscribe();
#### 检查或设置取消订阅状态 ####
Worker类的对象实现了Subscription接口，使用它的isUnsubscribed和unsubscribe方法，所以你可以在订阅取消时停止任务，或者从正在调度的任务内部取消订阅，示例：

    Worker worker = Schedulers.newThread().createWorker();
	Subscription mySubscription = worker.schedule(new Action0() {
	
	    @Override
	    public void call() {
	        while(!worker.isUnsubscribed()) {
	            status = yourWork();
	            if(QUIT == status) { worker.unsubscribe(); }
	        }
	    }
	
	});
Worker同时是Subscription，因此你可以（通常也应该）调用它的unsubscribe方法通知可以挂起任务和释放资源了。
#### 延时和周期调度器 ####
你可以使用schedule(action,delayTime,timeUnit)在指定的调度器上延时执行你的任务，下面例子中的任务将在500毫秒之后开始执行：

    someScheduler.schedule(someAction, 500, TimeUnit.MILLISECONDS);
使用另一个版本的schedule，schedulePeriodically(action,initialDelay,period,timeUnit)方法让你可以安排一个定期执行的任务，下面例子的任务将在500毫秒之后执行，然后每250毫秒执行一次：

    someScheduler.schedulePeriodically(someAction, 500, 250, TimeUnit.MILLISECONDS);
#### 测试调度器 ####
TestScheduler让你可以对调度器的时钟表现进行手动微调。这对依赖精确时间安排的任务的测试很有用处。这个调度器有三个额外的方法：

- advanceTimeTo(time,unit) 向前波动调度器的时钟到一个指定的时间点
- advanceTimeBy(time,unit) 将调度器的时钟向前拨动一个指定的时间段
- triggerActions( ) 开始执行任何计划中的但是未启动的任务，如果它们的计划时间等于或者早于调度器时钟的当前时间

线程调度器（Scheduler）是将RxJava从同步观察者模式转到异步观察者模式的一个重要工具，有了它才能更好的将RxJava应用到项目中。
## 参考： ##
[https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)

[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_1](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_1)

[http://blog.csdn.net/lzyzsd/article/details/44094895](http://blog.csdn.net/lzyzsd/article/details/44094895)

[http://www.ithao123.cn/content-9344110.html](http://www.ithao123.cn/content-9344110.html)

[http://blog.csdn.net/tangxl2008008/article/details/51334295](http://blog.csdn.net/tangxl2008008/article/details/51334295)

[http://www.jianshu.com/p/129a26c8ba9e](http://www.jianshu.com/p/129a26c8ba9e)

[https://github.com/mcxiaoke/RxDocs/blob/master/Scheduler.md](https://github.com/mcxiaoke/RxDocs/blob/master/Scheduler.md)