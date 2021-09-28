# 前言

![常用的设计模式的关系](https://gitee.com/huawesome/my-picture/raw/master/img/202109271307593.jpg)

# 工厂模式

Spring使用工厂模式可以通过 `BeanFactory` 或 `ApplicationContext` 创建 bean 对象。

# 单例模式

## Java中的单例模式

什么是单例，一个类只有一个实例，例如GC垃圾收集器。

单例模式分为懒汉式和饿汉式。

==饿汉式是线程安全的==在类初始化时就实例化了一个静态对象；==懒汉式是线程不安全的==在调用时才创建，在并发环境下可能创建多个对象

### 示例

#### 懒汉式

```java
public class Singleton {
	
	/**
	 * 该函数限制用户主动创建实例
	 */
	private Singleton() {}
	private static Singleton singleton = null;
	
	/**
	 * 获取Singleton实例（也叫静态工厂方法）
	 * @return Singleton
	 */
	public static Singleton getSingleton() {
		/* 当singleton为空时创建它，反之直接返回，保证唯一性 */
		if(singleton == null){
			singleton = new Singleton();
		}
		return singleton;
	}
	
}
```

线程安全的懒汉式：在`getSingleton()`添加`Synchronized`同步

```java
public class Singleton {
	
/**
	 * 该函数限制用户主动创建实例
	 */
	private Singleton() {}
	private static Singleton singleton = null;
	/**
	 * 获取Singleton实例，也叫静态工厂方法
	 * @return Singleton
	 */
	public static synchronized Singleton getSingleton(){
		if(singleton==null){
			singleton=new Singleton();
		}
		return singleton;
	}
	
}
```

双重检查锁定

```java
public class Singleton {

	/**
	 * 该函数限制用户主动创建实例
	 */
	private Singleton() {}

	private volatile static Singleton singleton = null;

	/**
	 * 获取Singleton实例，也叫静态工厂方法
	 * @return Singleton
	 */
	public static Singleton getInstance() {
		if (singleton == null) {
			synchronized (Singleton.class) {
				if (singleton == null) {
					singleton = new Singleton();
				}
			}
		}
		return singleton;
	}

}
```



静态内部类：静态内部类比双重检查锁定和在getInstance()方法上加同步都要好，实现了线程安全又避免了同步带来的性能影响

```java
public class Singleton {

	/**
	 * 静态内部类
	 * @author kimball
	 *
	 */
	private static class LazyHolder {
		// 创建Singleton实例
		private static final Singleton INSTANCE = new Singleton();
	}

	/**
	 * 该函数限制用户主动创建实例
	 */
	private Singleton() {}

	/**
	 * 获取Singleton实例，也叫静态工厂方法
	 * @return Singleton
	 */
	public static final Singleton getInstance() {
		return LazyHolder.INSTANCE;
	}

}
```

Java机制规定，内部类 LazyHolder只有在getInstance()方法第一次调用的时候才会被加载（实现了延迟加载效果），而且其加载过程是线程安全的（实现线程安全）

#### 饿汉式

```java
public class Singleton {

	/**
	 * 该函数限制用户主动创建实例
	 */
	private Singleton() {}

	private static final Singleton singleton = new Singleton();

	/**
	 * 获取Singleton实例，也叫静态工厂方法
	 * @return Singleton
	 */
	public static Singleton getInstance() {
		return singleton;
	}

}
```

### 参考

[Java设计模式(一)-单例模式](https://zhuanlan.zhihu.com/p/23713957)

## Spring中的单例

Spring中Bean的默认作用域就是singleton。

Spring Bean的一级缓存就是单例的例子，代码如下：

```java
// 通过 ConcurrentHashMap（线程安全） 实现单例注册表
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // 检查缓存中是否存在实例  
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //...省略了很多代码
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //...省略了很多代码
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    //将对象添加到单例注册表
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

还有Spring依赖注入中，使用了双重判断加锁的实例模式。[地址](https://zhuanlan.zhihu.com/p/80928209)

# 代理模式

AOP中的JDK动态代理和CGLIB动态代理

# 模板方法

Spring JDBC

# 观察者模式

Spring 事件驱动模型就是观察者模式很经典的一个应用。

# 适配器模式

Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

# 装饰者模式

我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

