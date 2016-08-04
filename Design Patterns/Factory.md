# 工厂模式 #
工厂方法模式（Factory Pattern）是创建型设计模式之一。工厂方法模式是一种结构简单的模式，其在我们平时开发中应用很广泛，也许你不知道，但是你已经使用了无数次改模式了。例如Activity中的各个生命周期方法，以onCreate()为例，它就可以看作是一个工厂方法，我们可以在其中构造我们的View并通过setContentView()返回给framework处理。
## 工厂模式的定义和使用场景 ##
工厂模式用来定义一个用于创建对象的接口，让子类决定实例化哪个类。在任何需要生成复杂对象的地方都可以使用工厂方法模式。工厂模式的UML类图如下图所示
![](http://i.imgur.com/V4bAS8v.png)
根据上图我们可以得到一个工厂方法模式的通用模式代码。
1. 抽象产品类

	    publib abstract class Product {
			/**
			*产品类的抽象方法
			*由具体的产品类去实现
			*/
			public abstract void method();
		}

2. 具体产品类A

	    public class ConcreteProductA extends Product {
			@Override
			public void method(){
				System.out.println("This is ConcreteProductA");
			}
		}

3. 具体产品类B

	    public class ConcreteProductB extends Product {
			@Override
			public void method(){
				System.out.println("This is ConcreteProductB");
			}
		}

4. 抽象工厂类

	    publib abstract class Factory {
			/**
			*抽象工厂方法
			*具体生成什么由子类去实现
			*/
			public abstract Product createProduct();
		}	

5. 具体工厂类

	    publib class ConcreteFactory extends Factory {
			@Override
			public Product createProduct(){
				return new ConcreteProductA();
			}
		}	

6. 客户类

	    publib class Client {
			public static void main(String[] args) {
				Factory factory = new ConcreteFactory();
				Product product = factory.createProduct();
				product.method();
			}
		}	

该示例的输出为：

    This is ConcreteProductA.
## 反射实现工厂模式 ##
利用反射的方式来生产具体产品对象会更简洁，我们只需要在工厂方法的参数列表中传入一个Class类来决定是哪一个产品类就可以生产出对应的对象。
1. 抽象工厂类

	    publib abstract class Factory {
			/**
			*抽象工厂方法
			*具体生成什么由子类去实现
			*
			* @param clz 产品对象类类型
			* 
			* @return 具体的产品对象
			*/
			public abstract <T extends Product> T createProduct(Class<T> clz);
		}

2. 具体工厂类

	    publib class ConcreteFactory extends Factory {
			@Override
			public <T extends Product> T createProduct(Class<T> clz){
				Product p = null;
				try{
					p = (Product) Class.forName(clz.getName()).newInstance();
				}catch (Exception e){
					e.printStackTrace();
				}
				return (T) p;
			}
		}	

3. 客户类

	    publib class Client {
			public static void main(String[] args) {
				Factory factory = new ConcreteFactory();
				Product product = factory.createProduct(ConcreteProductA.class);
				product.method();
			}
		}	

该示例的输出同样为：

    This is ConcreteProductA.
## 总结##
总的来说，工厂方法模式是一个很好的设计模式，便于在需要生成多个复杂对象的时候进行使用并且该模式的实现过程较为简单，比较好掌握。