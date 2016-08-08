# IPC机制学习笔记 #
IPC是Inter-Process Communication的缩写，含义为进程间通信或者是跨进程通信，是指两个进程之间进行数据交换的过程。在操作系统中定义，线程是CPU调度的最小单元，同时线程是一种有限的系统资源。而进程一般指一个执行单元，在PC和移动设备上是指一个程序或者是一个应用。
## 多进程和多进程产生的问题 ##
多进程的情况主要分为两种，一种是一个应用因为某些原因自身需要采用多进程模式实现，第二种是为了加大一个应用可使用的内存所以需要通过多进程来获取多份内存空间。正常情况下，Android中多进程是指一个应用中存在多个进程的情况。在Android中使用多进程基本上只有一种方法，那就是给四大组件（Activity、Service、Receiver、ContentProvider）在AndroidManifest中指定android:process属性，有两种方式，如下所示

    <activity
		android:name="com.example.activity.IPCActivity"
		android:process=":remote"/>
	<activity
		android:name="com.example.activity.IPCActivity1"
		android:process="com.example.activity.remote"/>
注意：进程名以“:”开头的进程属于当前应用的私有进程，其他应用的组件不可以和它跑在同一个进程中，而进程名不以“:”开头的进程属于全局进程，其他应用通过ShareUID方式可以和它跑在同一个进程中。
Android为每一个应用分配了一个独立的虚拟机，或者说是为每个进程都分配了一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这就导致在不同虚拟机中访问同一个类的对象会产生多份副本。正是由于这个原因，使用多进程会造成以下几方面的问题：
1. 静态成员和单例模式完全失效
2. 线程同步机制完全失效
3. SharedPreference的可靠性下降
4. Application会多次创建

## Binder机制 ##
Binder机制是Android中的一种IPC通信机制。它的设计是基于C/S结构，其中一个进程作为Server端，提供服务；多个进程作为Client端，借助Binder与Server端完成通信获取服务。Binder可以看成是Server端提供的某个特定服务的访问接入点，Client通过这个接入点来获取该服务。Binder的实体对象位于Server端，该对象的引用位于Client端，CLient通过Binder引用来访问Server。关于Binder机制较为详细的分析，请移步之前的博文[《Binder学习笔记》](http://blog.csdn.net/siguoyi/article/details/52084813)，在此我只是简单的介绍一下。Binder机制可以通过下图来描绘
![](http://i.imgur.com/TMw7c4C.png)
具体的过程如下：通信过程中，ServiceManager首先将字符型的Binder名字转换为Binder对象的引然后Client向Server端发起一个远程服务请求并将当前线程挂起（所以需要在非UI线程中发起请求），Server端的Binder实体对象将参数传入对应服务中，并在线程池中运行服务，得到结果后将结果返回给Client并唤醒挂起的线程完成进程间通信。
## Android中的IPC方式 ##
在Android中有多种实现进程间通信的方式：Bundle，文件共享，Messenger，AIDL，ContentProvider，Socket。接下来，我会对每种方式进行相应的介绍。

1. 使用Bundle
四大组件中的三大组件（Activity，Service和Receiver）都是支持在Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以它可以方便的在不同的进程间传输。我们在传输过程中，传输的数据必须能够被序列化，如基本类型，实现了Parcelable接口的对象，实现了Serializable接口的对象以及一些Android支持的特殊对象（即Bundle支持的类型）。
2. 使用文件共享
共享文件也是一种不错的进程间通信的方式，两个进程通过读/写同一个文件来交换数据，因为Android是机遇Linux系统的，所以Android读写文件可以没有限制的进行，两个线程同时对一个文件进行写操作都是允许的（最好做好同步）。除了可以交换一些文本信息外，我们还可以在序列化一个对象到文件系统中的同时在另外一个进程中恢复这个对象。保存的文件格式可以是文本文件也可以是XML文件，只要读/写双方约定数据格式即可，文件共享方式适合在对数据同步要求不高的进程之间进行通信。注意：不建议在进程间通信时使用SharedPreferences。
3. 使用Messenger
Messenger可以理解为信使，顾名思义，通过它可以在不同的进程中传递Message对象，在Message对象中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递。Messenger是一种轻量级的IPC方案，它的底层实现是AIDL。Messenger实现进程间通信的原理如下图所示：
![](http://i.imgur.com/ly6ZVYI.png)
Messenger实现进程间通信的过程如下：
	- 首先，在Server端创建一个Service来处理Client的连接请求，同时创建一个handler对象，并用这个Handler对象创建一个Messenger对象，然后在Service的onBind方法中返回这个Messenger对象底层的Binder即可。
	- Client端在与Server端建立连接之后在onServiceConnected方法中获取到Binder，并用这个Binder创建一个Messenger对象，将需要传递的信息及封装到Message对象里面，通过Messenger对象将消息发送给Server。
	- Server端接受到消息之后，用Message.replyTo创建Messenger对象，通过这个Messenger对象将消息发送给Client，完成一次完整的进程间C/S通信。
4. 使用AIDL
在上一节中介绍了使用Messenger进行进程间通信的方法，Messenger是以串行的方式来处理客户端发来的信息，一旦信息量过大，服务端将会发生阻塞。这个时候我们可以采用AIDL来实现并行的跨进程方法调用。
AIDL实现进程间通信的过程如下;
	- 服务端首先要创建一个Service用来监听客户端的连接请求，然后创建一个AIDL文件，将暴露给客户端的接口在这个AIDL文件中声明，最后在Service中实现这个AIDL接口即可。
	- Client端首先需要绑定服务端的Service，绑定成功后，将服务端返回的Binder对象转换成AIDL接口所属类型，接着就可以调用AIDL中的方法了。
5. 使用ContentProvider
具体使用方式可以参照这篇博文[http://blog.csdn.net/chuyuqing/article/details/39995607](http://blog.csdn.net/chuyuqing/article/details/39995607)
6. 使用Socket
具体使用方式与Android中的Socket编程几乎一致，可以参考此博文[http://blog.csdn.net/x605940745/article/details/17001641](http://blog.csdn.net/x605940745/article/details/17001641)

## Binder连接池 ##
AIDL是最常用的进程间通信方式之一。AIDL的使用流程如下：首先创建一个Service和一个AIDL接口，接着创建一个类继承自AIDL接口中的Stub类并实现Stub中的抽象方法，在Service的onBind方法中返回这个类的对象，然后再客户端就可以绑定服务端Service，建立连接后就可以访问远程服务端的方法了。

如果在项目中有很多业务模块都需要用AIDL来进行进程间通信，这样就会创建很多的Service，然而Service作为一个远程服务同时又是一种系统资源，创建太多会造成资源浪费，所以我们应该避免过多地创建Service。在这种情况下，Binder连接池应运而生，它的主要作用就是将每个业务模块的Binder请求统一转发给一个远程Service中去执行，在Service中通过每个业务模块的标识ID来执行对应的方法，从而避免了重复创建Service的过程。可以在Application中提前对Binder连接池进行初始化，推荐大家在AIDL开发中引入连接池BinderPool机制。
## IPC方式的优缺点和适用场景 ##
![](http://i.imgur.com/VbGoMxV.png)
## 总结 ##
IPC在Android开发中是一个比较重要的部分，而且在Android面试的过程中也经常问到，所以有必要去把它搞清楚。在平时的开发中，在不经意间已经使用过了ContentProvider和Socket这两种方式，剩下的几种方式可以在今后的开发中去尝试使用。
## 参考 ##
[http://blog.csdn.net/luoshengyang/article/details/6618363](http://blog.csdn.net/luoshengyang/article/details/6618363)

[http://blog.csdn.net/hitlion2008/article/details/9773251](http://blog.csdn.net/hitlion2008/article/details/9773251)

[http://blog.csdn.net/chaihuasong/article/details/10071903](http://blog.csdn.net/chaihuasong/article/details/10071903)