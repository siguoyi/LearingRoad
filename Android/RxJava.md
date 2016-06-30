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
## 参考： ##
[https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)

[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_1](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1012/3572.html#toc_1)
