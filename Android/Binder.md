# Binder学习笔记 #
在Android系统中，每一个应用程序都是由一些Activity和Service组成的，这些Activity和Service有可能运行在同一个进程中，也有可能运行在不同的进程中。那么，不在同一个进程的Activity或者Service是如何通信的呢？这就是本文中要介绍的Binder进程间通信机制了。

Binder是一种进程间通信机制，它是一种类似于COM和CORBA分布式组件架构，通俗一点，其实是提供远程过程调用（RPC）功能。从英文字面上意思看，Binder具有粘结剂的意思，那么它把什么东西粘结在一起呢？在Android系统的Binder机制中，由一系统组件组成，分别是Client、Server、Service Manager和Binder驱动程序，其中**Client、Server和Service Manager运行在用户空间，Binder驱动程序运行在内核空间**。Binder就是一种把这四个组件粘合在一起的粘结剂了，其中，核心组件便是Binder驱动程序了，Service Manager提供了辅助管理的功能，Client和Server正是在Binder驱动和Service Manager提供的基础设施上，进行Client-Server之间的通信。Service Manager和Binder驱动已经在Android平台中实现好，开发者只要按照规范实现自己的Client和Server组件就可以了。说起来简单，做起难，对初学者来说，Android系统的Binder机制是最难理解的了，而Binder机制无论从系统开发还是应用开发的角度来看，都是Android系统中最重要的组成，因此，很有必要深入了解Binder的工作方式。要深入了解Binder的工作方式，最好的方式莫过于是阅读Binder相关的源代码了，Linux的鼻祖Linus Torvalds曾经曰过一句名言RTFSC：Read The Fucking Source Code。

## 为什么要用Binder ##
Linux已经拥有管道，system V IPC，socket等IPC手段，却还要依赖Binder来实现进程间通信，说明Binder具有无可比拟的优势。

目前linux支持的IPC包括传统的管道，System V IPC，即消息队列/共享内存/信号量，以及socket中只有socket支持Client-Server的通信方式。当然也可以在这些底层机制上架设一套协议来实现Client-Server通信，但这样增加了系统的复杂性，在手机这种条件复杂，资源稀缺的环境下可靠性也难以保证。

另一方面是传输性能。socket作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程。共享内存虽然无需拷贝，但控制复杂，难以使用。

|  IPC  |  数据拷贝次数  |
| -- | ---------- |
| 共享内存    | 0 |
| Binder    | 1 |
| Socket/管道/消息队列 | 2 |

还有一点是出于安全性考虑。Android为每个安装好的应用程序分配了自己的UID，故进程的UID是鉴别进程身份的重要标志。使用传统IPC只能由用户在数据包里填入UID和PID，但这样不可靠，容易被恶意程序利用。可靠的身份标记只有由IPC机制本身在内核中添加。其次传统IPC访问接入点是开放的，无法建立私有通道。比如命名管道的名称，systemV的键值，socket的ip地址或文件名都是开放的，只要知道这些接入点的程序都可以和对端建立连接，不管怎样都无法阻止恶意程序通过猜测接收方地址获得连接。

基于以上原因，Android需要建立一套新的IPC机制来满足系统对通信方式，传输性能和安全性的要求，这就是Binder。**Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送发添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。**

## 面向对象的Binder IPC ##
Binder使用Client-Server通信方式：一个进程作为Server提供诸如视频/音频解码，视频捕获，地址本查询，网络连接等服务；多个进程作为Client向Server发起服务请求，获得所需要的服务。要想实现Client-Server通信据必须实现以下两点：一是server必须有确定的访问接入点或者说地址来接受Client的请求，并且Client可以通过某种途径获知Server的地址；二是制定Command-Reply协议来传输数据。例如在网络通信中Server的访问接入点就是Server主机的IP地址+端口号，传输协议为TCP协议。

对Binder而言，Binder可以看成Server提供的实现某个特定服务的访问接入点， Client通过这个‘地址’向Server发送请求来使用该服务；对Client而言，Binder可以看成是通向Server的管道入口，要想和某个Server通信首先必须建立这个管道并获得管道入口。

Binder的实体对象位于Server中，该对象的引用位于Client中。Client通过Binder的引用访问Server。而软件领域另一个术语“句柄”也可以用来表述Binder在Client中的存在方式。从通信的角度看，Client中的Binder也可以看作是Server Binder的“代理”，在本地代表远端Server为Client提供服务。本文中会使用“引用”或“句柄”这个两广泛使用的术语。

面向对象思想的引入**将进程间通信转化为通过对某个Binder对象的引用调用该对象的方法**，而其独特之处在于Binder对象是一个可以跨进程引用的对象，它的实体位于一个进程中，而它的引用却遍布于系统的各个进程之中。最诱人的是，这个引用和java里引用一样既可以是强类型，也可以是弱类型，而且可以从一个进程传给其它进程，让大家都能访问同一Server，就象将一个对象或引用赋值给另一个引用一样。Binder模糊了进程边界，淡化了进程间通信过程，整个系统仿佛运行于同一个面向对象的程序之中。形形色色的Binder对象以及星罗棋布的引用仿佛粘接各个应用程序的胶水，这也是Binder在英文里的原意。

## Binder的通信模型 ##
Binder框架定义了四个角色：Server，Client，ServiceManager（以后简称SMgr）以及Binder驱动。其中Server，Client，SMgr运行于用户空间，驱动运行于内核空间。这四个角色的关系和互联网类似：Server是服务器，Client是客户终端，SMgr是域名服务器（DNS），驱动是路由器。
### Binder驱动 ###
和路由器一样，Binder驱动虽然默默无闻，却是通信的核心。尽管名叫“驱动”，实际上和硬件设备没有任何关系，只是实现方式和设备驱动程序是一样的：它工作于内核态，提供open()，mmap()，poll()，ioctl()等标准文件操作，以字符驱动设备中的misc设备注册在设备目录/dev下，用户通过/dev/binder访问该它。**驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持**。驱动和应用程序之间定义了一套接口协议，主要功能由ioctl()接口实现，不提供read()，write()接口，因为ioctl()灵活方便，且能够一次调用实现先写后读以满足同步交互，而不必分别调用write()和read()。Binder驱动的代码位于linux目录的drivers/staging/android/binder.c中。
### ServiceManager 与实名Binder ###
和DNS类似，SMgr的作用是**将字符形式的Binder名字转化成Client中对该Binder的引用**，使得Client能够通过Binder名字获得对Server中Binder实体的引用。注册了名字的Binder叫实名Binder，就象每个网站除了有IP地址外还有自己的网址。Server创建了Binder实体，为其取一个字符形式，可读易记的名字，将这个Binder连同名字以数据包的形式通过Binder驱动发送给SMgr，通知SMgr注册一个名叫张三的Binder，它位于某个Server中。驱动为这个穿过进程边界的Binder创建位于内核中的实体节点以及SMgr对实体的引用，将名字及新建的引用打包传递给SMgr。SMgr收数据包后，从中取出名字和引用填入一张查找表中。

细心的读者可能会发现其中的蹊跷：SMgr是一个进程，Server是另一个进程，Server向SMgr注册Binder必然会涉及进程间通信。当前实现的是进程间通信却又要用到进程间通信，这就好象蛋可以孵出鸡前提却是要找只鸡来孵蛋。Binder的实现比较巧妙：预先创造一只鸡来孵蛋：**SMgr和其它进程同样采用Binder通信，SMgr是Server端，有自己的Binder对象（实体），其它进程都是Client，需要通过这个Binder的引用来实现Binder的注册，查询和获取**。SMgr提供的Binder比较特殊，它没有名字也不需要注册，当一个进程使用**BINDER_SET_CONTEXT_MGR**命令将自己注册成SMgr时Binder驱动会自动为它创建Binder实体（这就是那只预先造好的鸡）。其次这个Binder的引用在所有Client中都**固定为0而无须通过其它手段获得**。也就是说，一个Server若要向SMgr注册自己Binder就必需通过0这个引用号和SMgr的Binder通信。类比网络通信，0号引用就好比域名服务器的地址，你必须预先手工或动态配置好。要注意这里说的Client是相对SMgr而言的，一个应用程序可能是个提供服务的Server，但对SMgr来说它仍然是个Client。

### Client 获得实名Binder的引用 ###
Server向SMgr注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的引用了。**Client也利用保留的0号引用向SMgr请求访问某个Binder**：我申请获得名字叫张三的Binder的引用。SMgr收到这个连接请求，从请求数据包里获得Binder的名字，在查找表里找到该名字对应的条目，从条目中取出Binder的引用，将该引用作为回复发送给发起请求的Client。从面向对象的角度，这个Binder对象现在有了两个引用：一个位于SMgr中，一个位于发起请求的Client中。如果接下来有更多的Client请求该Binder，系统中就会有更多的引用指向该Binder，就象java里一个对象存在多个引用一样。而且类似的这些指向Binder的引用是强类型，从而确保只要有引用Binder实体就不会被释放掉。通过以上过程可以看出，SMgr象个火车票代售点，收集了所有火车的车票，可以通过它购买到乘坐各趟火车的票--得到某个Binder的引用。
### 匿名 Binder ###
并不是所有Binder都需要注册给SMgr广而告之的。Server端可以通过已经建立的Binder连接将创建的Binder实体传给Client，当然这条已经建立的Binder连接**必须是通过实名Binder实现**。由于这个Binder没有向SMgr注册名字，所以是个匿名Binder。Client将会收到这个匿名Binder的引用，通过这个引用向位于Server中的实体发送请求。匿名Binder为通信双方建立一条私密通道，只要Server没有把匿名Binder发给别的进程，别的进程就无法通过穷举或猜测等任何方式获得该Binder的引用，向该Binder发送请求。
## Binder使用示例 ##
下面我们通过一个简单的例子来看一下，如何通过Binder访问远程Service

首先，我们需要创建一个RemoteServiceTestActivity作为测试RemoteService的Activity，binder.xml为该Activity的布局文件。布局文件的代码如下：

    <?xml version="1.0" encoding="utf-8"?>
	<LinearLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:gravity="center"
	    android:orientation="vertical">
	
	    <TextView
	        android:id="@+id/tv_binder"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/binder_test"
	        android:textSize="30sp"
	        android:textStyle="bold|italic"
	        android:textColor="#8b04ac"/>
	
	    <Button
	        android:id="@+id/btn_bind"
	        android:layout_width="100dp"
	        android:layout_height="wrap_content"
	        android:layout_marginTop="10dp"
	        android:text="@string/bind"
	        android:textAllCaps="false"
	        android:textSize="20sp"/>
	
	</LinearLayout>
其中用到的string.xml资源如下：

    <string name="binder_test">Binder Test</string>
    <string name="bind">Bind</string>
接下来是RemoteServiceTestActivity.java的代码

    public class RemoteServiceTestActivity extends AppCompatActivity {
	    public static final String TAG = "RemoteServiceTest";
	    @BindView(R.id.tv_binder)
	    TextView tvBinder;
	    @BindView(R.id.btn_bind)
	    Button btnBind;
	    private ServiceConnection serviceConnection;
	    public static final int SAY_HELLO_TO_CLIENT = 0;
	
	    @OnClick(R.id.btn_bind)
	    public void onClick() {
	        Intent service = new Intent(this.getApplicationContext(), RemoteService.class);
	        this.bindService(service, serviceConnection, Context.BIND_AUTO_CREATE);
	    }
	
	    /**
	     * Handler of incoming messages from service.
	     */
	    class IncomingHandler extends Handler {
	        @Override
	        public void handleMessage(Message msg) {
	            switch (msg.what) {
	                case SAY_HELLO_TO_CLIENT:
	                    tvBinder.setText("Hello World Remote Client!");
	                    Toast.makeText(RemoteServiceTestActivity.this.getApplicationContext(), "Hello World Remote Client!",
	                            Toast.LENGTH_SHORT).show();
	                    break;
	                default:
	                    super.handleMessage(msg);
	            }
	        }
	    }
	
	    Messenger messenger_reciever = new Messenger(new IncomingHandler());
	
	    /**
	     * Called when the activity is first created.
	     */
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_binder);
	        ButterKnife.bind(this);
	
	        serviceConnection = new ServiceConnection() {
	            @Override
	            public void onServiceConnected(ComponentName name, IBinder service) {
	                Log.d(TAG, "service connected");
	                Messenger messenger = new Messenger(service);
	                Message msg = new Message();
	                msg.what = RemoteService.MSG_SAY_HELLO;
	                msg.replyTo = messenger_reciever;
	                try {
	                    messenger.send(msg);
	                } catch (RemoteException e) {
	                    e.printStackTrace();
	                }
	            }
	
	            @Override
	            public void onServiceDisconnected(ComponentName name) {
	                Log.d(TAG, "service disconnected");
	            }
	        };
	    }
	
	    @Override
	    protected void onStart() {
	        super.onStart();
	    }
	
	    @Override
	    protected void onStop() {
	        super.onStop();
	        //must unbind the service otherwise the ServiceConnection will be leaked.
	        this.unbindService(serviceConnection);
	    }
	}
在创建好了RemoteServiceTestActivity之后，我们需要创建一个远程服务RemoteService

    public class RemoteService extends Service {
	    public static final int MSG_SAY_HELLO = 0;
	
	    @Nullable
	    @Override
	    public IBinder onBind(Intent intent) {
	        return messager.getBinder();
	    }
	
	    Handler IncomingHandler = new Handler() {
	
	        @Override
	        public void handleMessage(Message msg) {
	            if(msg.replyTo != null){
	                Message msg_client = this.obtainMessage();
	                msg_client.what = RemoteServiceTestActivity.SAY_HELLO_TO_CLIENT;
	                try {
	                    ((Messenger)msg.replyTo).send(msg_client);
	                } catch (RemoteException e) {
	                    // TODO Auto-generated catch block
	                    e.printStackTrace();
	                }
	            }
	            switch (msg.what) {
	                case MSG_SAY_HELLO:
	                    Toast.makeText(RemoteService.this.getApplicationContext(), "Hello World Remote Service!",
	                            Toast.LENGTH_SHORT).show();
	                    break;
	                default:
	                    super.handleMessage(msg);
	            }
	        }
	
	    };
	
	    Messenger  messager = new Messenger(IncomingHandler);
	}
最后在AndroidManifest.xml文件中注册相应的组件，需要注明RemoteService的process为remote

    <activity android:name=".activity.RemoteServiceTestActivity"/>

	<service android:name=".binder.RemoteService" android:process=":remote"/>
运行Demo，点击Bind按钮就可以通过binder进行远程Service的绑定，在绑定成功后想RemoteService发送一条Messenger消息

    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        Log.d(TAG, "service connected");
        Messenger messenger = new Messenger(service);
        Message msg = new Message();
        msg.what = RemoteService.MSG_SAY_HELLO;
        msg.replyTo = messenger_reciever;
        try {
            messenger.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
RemoteService在收到Messenger消息后，判断Messenger.reply不为null，则会给Client发送一条Messenger消息

    if(msg.replyTo != null){
        Message msg_client = this.obtainMessage();
        msg_client.what = RemoteServiceTestActivity.SAY_HELLO_TO_CLIENT;
        try {
            ((Messenger)msg.replyTo).send(msg_client);
        } catch (RemoteException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
另外，RemoteService还会处理接受到的Messenger消息

    switch (msg.what) {
	    case MSG_SAY_HELLO:
	        Toast.makeText(RemoteService.this.getApplicationContext(), "Hello World Remote Service!",
	                Toast.LENGTH_SHORT).show();
	        break;
	    default:
	        super.handleMessage(msg);
	}
当Client端收到来自RemoteService回传的Messenger消息时，也会通过Handler的HandleMessage处理该消息

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case SAY_HELLO_TO_CLIENT:
                tvBinder.setText("Hello World Remote Client!");
                Toast.makeText(RemoteServiceTestActivity.this.getApplicationContext(), "Hello World Remote Client!",
                        Toast.LENGTH_SHORT).show();
                break;
            default:
                super.handleMessage(msg);
        }
    }
至此，我们完成了Client和RemoteService（也可理解为Server）的一次完整的交互过程。
## 总结 ##
Binder使用Client-Server通信方式，安全性好，简单高效，再加上其面向对象的设计思想，独特的接收缓存管理和线程池管理方式，成为Android进程间通信的中流砥柱。
## 参考 ##
[http://blog.csdn.net/luoshengyang/article/details/6618363/](http://blog.csdn.net/luoshengyang/article/details/6618363/)

[http://www.cnblogs.com/angeldevil/p/3296381.html](http://www.cnblogs.com/angeldevil/p/3296381.html)

[http://blog.csdn.net/universus/article/details/6211589](http://blog.csdn.net/universus/article/details/6211589)

