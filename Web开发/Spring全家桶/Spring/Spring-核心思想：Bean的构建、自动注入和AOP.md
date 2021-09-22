# Spring-核心思想：Bean的构建、自动注入和AOP

## 引言：“传统”Java编程的紧耦合问题

在进行“传统”Java编程时，对象与对象之间都是金密耦合的，例如服务类Service使用组件ComponentA，可能会写下面这样的代码：

```java
public class Service {
    private ComponentA component = new ComponentA("first Component");
}
```

在项目初期，这样应该没什么问题。

而随着项目的推进，这样的写法显然缺乏“拥抱变化”的能力。加入有一天，我们的ComponentsA的类的构造器需要更多的参数了，你会发现，有很多的地方需要修改。

那么这样写行不行呢？

```java
public class Service {
    private ComponentA component;
    public Service(ComponentA component){
        this.component = component;
    }
}
```

答案是不行的，因为在构建Service对象的时候，还是得使用new关键字来构建Component，需要修改的调用处并不少。

还有什么别的不好的地方吗？上面说的是非单例的情况，如果ComponentA本身是一个单例，会不会好点？毕竟我们可以找一个地方new一次ComponentA实例就够了，但是你可能会发下别的问题。

下面是一段用“双重检验锁”实现的ComponentA类：

```java
public class ComponentA{
  private volatile static ComponentA INSTANCE;
  
  private ComponentA(){}
  
  private static ComponentA getInstance*(){
    if (INSTANCE == null){
      synchronized (ComponentA.class){
        if (INSTANCE == null){
          INSTANCE = new ComponentA();
        }
      }
    }
    return INSTANCE;
  }
}
```

其实写了这么多代码，最终我们只是要一个单例而已。而且假设我们有ComponentB、ComponentC等，那么上面的重复性代码都得写一遍。

除了上述`代码中会出现大量功能一样的代码`的问题，还有不易于测试，不以扩展功能（例如支持AOP）等缺点。说白了，`所有问题的根源（之一）就是对象与对象之间耦合性太强了。`

## Spring是怎么解决这个问题的

这里套用一个租房的场景。我们为什么听过中介来租房子呢？因为省事呀，只要花点小钱就不能与房东产生直接的“纠缠”了。

`Spring就是这个思路，他就像一个“中介”公司`。当你需要一个依赖的对象（房子）时，你直接把你的需求告诉Spring（中介）就好了，它会帮你搞定这些依赖对象，按需创建它们，而无需你的任何额外操作。

不过，在Spring中，`房东和租客都是对象实例，只不过换了一个名字叫Bean`而已。

可以说通过一套稳定的生产流程，作为“中介”的Spring完成了生产和预装（牵线搭桥）这些Bean的任务。此时，你可能想了解更多。例如，如果一个Bean（租房者）需要用到另外一个Bean（房子）是，具体是怎么操作的呢？

本质上只能从Spring“中介”里去找，有时候我们直接根据名称（小区名）去找，有时候则根据类型（户型），各种方法不尽相同。你就把`Spring理解成一个Map型的公司`即可，实现如下：

```java
public class BeanFactory {
  private Map<String, Bean> beanMap = new HashMap<>();
  
  public Bean getBean(String key){
    return beanMap.get(key);
  }
}
```

如上代码所示，Bean所属公司提供了对于Map的操作来完成查找，找到Bean后装配给其他对象，这就是`依赖查找`、`自动注入的`过程。

## Bean的创建

那么回过头来，这些Bean又是怎么被创建的呢？

对于一个项目而言，不可避免会出现两种情况：一些对象是需要Spring来管理的，另外一些（例如项目中其它的类和以来的jar中的类）又不需要。所以我们得有一个办法去标识哪些是需要成为Spring Bean，因此各式各样的注解才应运而生，例如Component注解等。

## Bean的发现-自动注入

那么有了这些注解后，谁又来做“发现”它们的工作呢？直接配置自然不成问题，但是很明显“自动发现”更让人省心。此时，我们往往需要一个扫描器，可以模拟写一下这样的扫描器：

```java
public class AnnotationScan {
  
  // 通过扫描包来找到Bean
  void scan(String packages) {
    
  }
}
```

有了扫描器，我们就知道哪些类是需要成为Bean。

那么怎么实例化为一个Bean（也就是一个对象实例而已）呢？很明显，只能通过`反射`来做了。不过这里面的方式可能有多种：

- java.lang.Class.newInsance()
- java.lang.reflect.Constructor.newInstance()
- ReflectionFactory.newConstructorForSerialization()

`有了创建，有了装配，一个Bean才能成为自己想要的样子。`

## Bean的加工-AOP

而需求总是源源不断的，我们有时候想记录一个方法调用的性能，有时候我们又想在方法调用是输出统一的调用日志。诸如此类，我们肯定不想频繁再来个散弹式的修改。所以我们有了AOP，帮忙拦截方法调用，进行功能扩展。拦截谁呢？在Spring中自然就是Bean了。

其实AOP并不神奇，结合刚才的Bean（中介）工厂来讲，假设我们判断出一个Bean需要“增强”了，我们直接让它从工厂返回的时候，就使用一个代理对象作为返回不就可以了么？示例如下：

```java
public class BeanFactory{
  private Map<String, Bean> beanMap = new HaskMap<>();
  
  public Bean getBean(String key){
    //查询是否创建过
    Bean bean = beanMap.get(key);
    if(bean != null){
      return bean;
    }
    
    //创建一个bean
    Bean bean = createBean();
    //判断要不要AOP
    boolean needAop = judgeIfNeedAop(bean);
    try{
      if(needAop)
        //创建代理对象
        bean = createProxyObject(bean);
      return bean;
    }finally{
      beanMap.put(key,bean);
    }
  }
}
```

那么怎么知道一个对象要不要AOP？既然一个对象要AOP，它肯定被标记了一些“规则”，例如拦截某个类的某某方法，示例如下：

```java
@Aspect
@Service
public class AopConfig{
  @Around("execution(* com.spring.puzzle.ComponentA.execute())")
  public void recordPayPerformance(ProceedingJoinPoint joinPoint) throws Throwable{
    //
  }
}
```

这个时候很明显了，假设你的Bean名字是ComponentA，那么就应该返回ComponentA类型的代理对象了。至于这些规则是怎么建立起来的呢？你看到它上面使用的各种注解大概就能明白其中的规则了，无非就是`扫描注解，根据注解创建规则`。



## 参考

- 五分钟轻松了解Spring基础知识：https://time.geekbang.org/column/article/364664

