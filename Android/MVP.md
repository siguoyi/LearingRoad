#MVP学习模式笔记#
## 为什么要用MVP模式 ##
在Android开发的过程中，我们都会不知不觉的使用到MVC的框架模式，这是因为原生 Android 开发应该已经是一个基础的 MVC 框架。MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

对于经典的 Android MVC 框架来说，如果只是简单的应用，业务逻辑写到 Activity 下面并无太多问题，但一旦业务逐渐变得复杂起来，每个页面之间有不同的数据交互和业务交流时，activity 的代码就会急剧膨胀，代码就会变得可读性，维护性很差。所以我想学习一下Android官方推荐的MVP框架模式，看看这个模式的结构。
## 什么是MVP模式 ##
MVP全称Model View Presenter，MVP模式的结构图如下图所示：
![](http://i.imgur.com/DmGcy0E.png)
在MVP模式里通常包含4个要素：

1. **View:** 负责绘制UI元素、与用户进行交互(在Android中体现为Activity)；
2. **View interface:** 需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试；
3. **Model:** 负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合)；
4. **Presenter:** 作为View与Model交互的中间纽带，处理与用户交互的负责逻辑。

MVP与MVC最大的区别就在与将Model和View通过Presenter隔开了，不再允许其互相直接通信，而所有的消息都是通过Presenter这个中间人来传递，而这样做的目的主要是为了将数据和展示划出更明确的界限。
## MVP模式内部是如何工作的 ##
Presenter作为中间者，它是**同时拥有View和Model的引用**的，为了在它们之间起到桥梁作用，即Presenter会主动和View和Model进行通信。**Model和View必须是完全隔离的，不允许两者之间互相通信**，保持对彼此的不感知，这样的好处是你彻底将数据和展示分离来开，并且可以独立的为Model去做测试。

Model在三者中是独立性最高的，Model不应该拥有对View的引用，而且Model也不需要保存对Presenter的引用，**对于Presenter而已，Model只需要提供接口**，等着Presenter来调用时返回相应数据即可，这和经典MVC模式中是非常不同的，在MVC中Model在数据发送变化后，是需要发送广播来告之View去更新用户界面的，而在MVP中，Model是不应该去通知View，而是通知Presenter来间接的更新View的。

而Presenter和Model的关系也应该是**基于接口来通信**，这样才能把Model和Presenter的**耦合度也降到最低**，那么在需要改变Model内部实现，甚至彻底替换Model的时候，Presenter则是无需随之改变的。这样做带来的另一个好处就是你可以通过Mock一个Model来对Presenter以及View做模拟测试了，从而提高了可测试性。

那么View和Presenter的关系呢？View是需要拥有对Presenter的引用，但仅仅是为了将用户的操作和事件立即传递给Presenter，为了让View和Presenter耦合较低，View也只应该通过接口与Presenter通信，从而保证View是完全被动的，一方面它由用户的操作触发来和Presenter通信，另一方面它完全受Presenter控制，唯一需要做的事情就是如何展示数据。

**简要总结三者之间的关系是：View和Model之间没有联系，View通过接口向Presenter来传递用户操作，Model不主动和Presenter联系，被动的等着Presenter来调用其接口，Presenter通过接口和View/Model来联系。**

## 实现一个简单的MVP框架 ##
在了解了MVP的工作模式之后，我决定用代码来实现一个简单的MVP框架，加深对MVP模式的理解，框架图如下所示：
![](http://i.imgur.com/nkf4NqL.png)
我们从Model开始创建一个接口DataSource.class

    public interface DataSource {
	    String getStringFromRemote();
	    String getStringFromCache();
	}
接口里面就简单的定义了两个方法，在创建接口以后，我们自然要创建该接口的实现类DataSourceImpl.class

    public class DataSourceImpl implements DataSource {
	    @Override
	    public String getStringFromRemote() {
	        return "Hello";
	    }
	
	    @Override
	    public String getStringFromCache() {
	        return " world !";
	    }
	}
该类实现了DataSource接口并实现了getStringFromRemote()和getStringFromCache()方法。接下来我们写View相关的代码，首先为了实现Presenter与View的通信，需要创建一个接口MainView.class

    public interface MainView {
	    void onShowString(String str);
	}
然后，创建MVPActivity.class作为View显示依赖的Activity并实现MainView接口

    public class MVPActivity extends AppCompatActivity implements MainView{
	
	    @BindView(R.id.tv_mvp)
	    TextView tvMvp;
	
	    private MainPresenter mainPresenter;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState){
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_mvp);
	        ButterKnife.bind(this);
	
	        loadData();
	    }
	
	    private void loadData() {
	        mainPresenter = new MainPresenter();
	        mainPresenter.addDataListener(this);
	        mainPresenter.getString();
	    }
	
	    @Override
	    public void onShowString(String str) {
	        tvMvp.setText(str);
	    }
	}
大家可以看到我们在代码中加载数据时已经调用了MainPresenter相关的代码，它最为MVP的Presenter部分，是比较核心的部分，它的代码如下

    public class MainPresenter {
	    MainView mainView;
	    DataManager taskManager;
	
	    public MainPresenter(){
	        this.taskManager = new DataManager(new DataSourceImpl());
	    }
	
	    public MainPresenter addDataListener(MainView viewListener) {
	        this.mainView = viewListener;
	        return this;
	    }
	
	    public void getString() {
	        String str = taskManager.getShowContent();
	        mainView.onShowString(str);
	    }
	}
可以看到，在MainPresenter中我们通过addDataListener传入了MainView的引用，通过这个引用Presenter就可以调用MainView来进行UI操作。此外，DataManager是用来连接Presenter和Model的桥梁，通过它来从Model中管理（或者叫获取）数据，并将数据交由MainPresenter处理。至此，我们就实现了一个简单的MVP框架，该Demo的效果图如下
![](http://i.imgur.com/vRKW3AJ.png)
## MVP与Activity、Fragment的生命周期 ##
从前文的分析可以看出，MVP有很多的有点，例如易于维护，易于测试，松耦合，复用性高，健壮稳定，易于拓展等。但是由于Presenter经常性地需要执行一些耗时操作，例如网络请求。通常我们采用的是强引用的方式，一旦Presenter在等候网络请求的返回时Activity已经被销毁，由于Presenter持有Activity对象的强引用，导致Activity对象无法被回收，于是就发生了内存泄漏。

我们通过弱引用以及Activity和Fragment的生命周期来解决这个问题。主要思想就是在onDestroy()的方法中解除Presenter与Activity的引用，由于Activity的onDestroy()不是任何情况下都会被调用，所以采用弱引用是一个比较好的选择。
## 总结 ##
通过MVP相关内容的学习，基本上get了MVP框架模式的思想，也通过Demo来加深理解。从整体效果来说，MVP是开发过程中非常值得推荐的架构模式，它能够将各组件进行解耦，并且带来良好的可拓展性，可测试性，稳定性，可维护性，同时使得每个类型的职责相对单一，简单。在以后的开发过程中，可以尝试用MVP模式进行开发，亲身体验它的魅力。
## 参考 ##
[http://dev.qq.com/topic/5799d7844bef22a823b3ad44](http://dev.qq.com/topic/5799d7844bef22a823b3ad44)

[http://blog.csdn.net/vector_yi/article/details/24719873](http://blog.csdn.net/vector_yi/article/details/24719873)

[http://www.jianshu.com/p/50c7124f408e](http://www.jianshu.com/p/50c7124f408e)