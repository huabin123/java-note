# 前言

思考题：

1. 什么是循环依赖？
2. 什么情况下循环依赖可以被处理？
3. Spring是怎么解决循环依赖的？



# 什么是循环依赖

A依赖B，B依赖A

```java
@Component
public class A {
    // A中注入了B
 @Autowired
 private B b;
}

@Component
public class B {
    // B中也注入了A
 @Autowired
 private A a;
}
```

# 什么情况下循环依赖可以被处理

Spring解决循环依赖是有前置条件的

1. 出现循环依赖的Bean必须是单例；
2. 双方依赖注入的方式不能全是构造器注入的方式。(一个是构造器，一个是setter依然能被解决)

```java
@Component
public class A {
// @Autowired
// private B b;
 public A(B b) {

 }
}

@Component
public class B {

// @Autowired
// private A a;

 public B(A a){

 }
}
```

在上面的例子中，双方都是通过构造器的方式注入，这个时候循环依赖无法被解决，因为在调用b时b无法完成创建，在启动时Spring会直接报错

```sh
Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?
```

# Spring是如何解决的循环依赖

分两种情况：

1. 没有AOP
2. 有AOP

## 没有AOP的循环依赖

```java
@Component
public class A {
    // A中注入了B
 @Autowired
 private B b;
}

@Component
public class B {
    // B中也注入了A
 @Autowired
 private A a;
}
```

流程分析：

1. 创建A

    首先应该知道，Spring在创建Bean的过程中分为三步

    1. 实例化，new一个对象；
    2. 属性注入
    3. 初始化，执行aware接口中的方法，初始化方法，完成AOP代理。

    ![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109270812315.jpg)

    创建A的过程实际上就是调用getBean方法，这个方法有两层含义：

    1. 创建一个新的Bean；
    2. 从缓存中获取到已经被创建的对象。

    我们现在分析的是第一种情况（缓存中还没有A）

    ###### 调用getSingleton(beanName)

    ```java
    Object sharedInstance = getSingleton(beanName);
    ```

    这个方法又会调用`getSingleton(beanName, true)`

    ```java
    @Override
    public Object getSingleton(String beanName) {
       return getSingleton(beanName, true);
    }
    ```

`getSingleton(beanName, true)`这个方法实际上就是到缓存中尝试去获取Bean，整个缓存分为三级

1. singletonObjects，一级缓存，存储的是所有创建好了的单例Bean；
2. earlySingletonObjects，完成实例化，但是还未进行属性注入及初始化的对象；
3. singletonFactories，提前暴露一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象。

因为A是第一次被创建，所以不管哪个缓存中都没有A，因此会进入getSingleton的另外一个重载方法`getSingleton(beanName,singletonFactory)。`

###### 调用`getSingleton(beanName, true)`

这个方法就是用来创建Bean的，源码如下：

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {

            // ....
            // 省略异常处理及日志
            // ....

            // 在单例对象创建前先做一个标记
            // 将beanName放入到singletonsCurrentlyInCreation这个集合中
            // 标志着这个单例Bean正在创建
            // 如果同一个单例Bean多次被创建，这里会抛出异常
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                // 上游传入的lambda在这里会被执行，调用createBean方法创建一个Bean后返回
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            // ...
            // 省略catch异常处理
            // ...
            finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                // 创建完成后将对应的beanName从singletonsCurrentlyInCreation移除
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                // 添加到一级缓存singletonObjects中
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```

通过creatBean方法返回的Bean最终被放到了一级缓存，也就是单例池中。

那么到这里可以得出一个结论：以及缓存中存储的是已经被完全创建好了的单例Bean。

###### 调用addSingletonFactory方法

![image-20210927095132758](https://gitee.com/huawesome/my-picture/raw/master/img/202109270951806.png)

在完成Bean的实例化后，属性注入之前Spring将Bean包装秤一个工厂添加进了三级缓存中，对应源码如下：

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

这里只是添加了一个工厂，通过这个工厂（`ObjectFactory`）的`getObject`方法可以得到一个对象，而这个对象实际上就是通过`getEarlyBeanReference`这个方法创建的。那么，什么时候会去调用这个工厂的`getObject`方法呢？这个时候就要到创建B的流程了。

当A完成了实例化并添加进了三级缓存后，就要开始为A进行属性注入了，在注入时发现A依赖了B，那么这个时候Spring又会去`getBean(b)`，然后反射调用setter方法完成属性注入。

![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109270954594.jpg)

因为B需要注入A，所以在创建B的时候，又会去调用`getBean(a)`，这个时候就又回到之前的流程了，但是不同的是，之前的`getBean`是为了创建Bean，而此时再调用`getBean`不是为了创建了，而是要从缓存中获取，因为之前A在实例化后已经将其放入了三级缓存`singletonFactories`中，所以此时`getBean(a)`的流程就是这样子了

![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109270957679.jpg)

从这里我们可以看出，注入到B中的A是通过`getEarlyBeanReference`方法提前暴露出去的一个对象，还不是一个完整的Bean，那么`getEarlyBeanReference`到底干了啥了，我们看下它的源码

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

它实际上就是调用了后置处理器的`getEarlyBeanReference`，而真正实现了这个方法的后置处理器只有一个，就是通过`@EnableAspectJAutoProxy`注解导入的`AnnotationAwareAspectJAutoProxyCreator`。**也就是说如果在不考虑`AOP`的情况下，上面的代码等价于：**

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    return exposedObject;
}
```

**也就是说这个工厂啥都没干，直接将实例化阶段创建的对象返回了！所以说在不考虑`AOP`的情况下三级缓存有用嘛？讲道理，真的没什么用，只谈作用的话放在二级缓存就可以了**，

这个时候我们需要将整个创建A这个Bean的流程走完，如下图：

![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109271002306.jpg)

从上图中我们可以看到，虽然在创建B时会提前给B注入了一个还未初始化的A对象，但是在创建A的流程中一直使用的是注入到B中的A对象的引用，之后会根据这个引用对A进行初始化，所以B中提前注入了一个没有经过初始化的A类型对象不会有问题。

## 有AOP的循环依赖

之前我们已经说过了，在普通的循环依赖的情况下，三级缓存没有任何作用。三级缓存实际上跟Spring中的`AOP`相关，我们再来看一看`getEarlyBeanReference`的代码：

```text
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

如果在开启`AOP`的情况下，那么就是调用到`AnnotationAwareAspectJAutoProxyCreator`的`getEarlyBeanReference`方法，对应的源码如下：

```java
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = getCacheKey(bean.getClass(), beanName);
    this.earlyProxyReferences.put(cacheKey, bean);
    // 如果需要代理，返回一个代理对象，不需要代理，直接返回当前传入的这个bean对象
    return wrapIfNecessary(bean, beanName, cacheKey);
}
```

回到上面的例子，我们对A进行了`AOP`代理的话，那么此时`getEarlyBeanReference`将返回一个代理后的对象，而不是实例化阶段创建的对象，这样就意味着B中注入的A将是一个代理对象而不是A的实例化阶段创建后的对象。

![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109271006521.jpg)

1. 在给B注入的时候为什么要注入一个代理对象？

    答：当我们对A进行了`AOP`代理时，说明我们希望从容器中获取到的就是A代理后的对象而不是A本身，因此把A当作依赖进行注入时也要注入它的代理对象

2. 明明初始化的时候是A对象，那么Spring是在哪里将代理对象放入到容器中的呢？

    ![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109271008435.jpg)

    在完成初始化后，Spring又调用了一次`getSingleton`方法，这一次传入的参数又不一样了，false可以理解为禁用三级缓存，前面图中已经提到过了，在为B中注入A时已经将三级缓存中的工厂取出，并从工厂中获取到了一个对象放入到了二级缓存中，所以这里的这个`getSingleton`方法做的时间就是从二级缓存中获取到这个代理后的A对象。`exposedObject == bean`可以认为是必定成立的，除非你非要在初始化阶段的后置处理器中替换掉正常流程中的Bean，例如增加一个后置处理器：

    ```java
    @Component
    public class MyPostProcessor implements BeanPostProcessor {
     @Override
     public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      if (beanName.equals("a")) {
       return new A();
      }
      return bean;
     }
    }
    ```

    不过，请不要做这种骚操作，徒增烦恼！

3. 初始化的时候是对A对象本身进行初始化，而容器中以及注入到B中的都是代理对象，这样不会有问题吗？

    答：不会，这是因为不管是`cglib`代理还是`jdk`动态代理生成的代理类，内部都持有一个目标类的引用，当调用代理对象的方法时，实际会去调用目标对象的方法，A完成初始化相当于代理对象自身也完成了初始化

4. 三级缓存为什么要使用工厂而不是直接使用引用？换而言之，为什么需要这个三级缓存，直接通过二级缓存暴露一个引用不行吗？

    答：**这个工厂的目的在于延迟对实例化阶段生成的对象的代理，只有真正发生循环依赖的时候，才去提前生成代理对象，否则只会创建一个工厂并将其放入到三级缓存中，但是不会去通过这个工厂去真正创建对象**

    我们思考一种简单的情况，就以单独创建A为例，假设AB之间现在没有依赖关系，但是A被代理了，这个时候当A完成实例化后还是会进入下面这段代码：

    ```text
    // A是单例的，mbd.isSingleton()条件满足
    // allowCircularReferences：这个变量代表是否允许循环依赖，默认是开启的，条件也满足
    // isSingletonCurrentlyInCreation：正在在创建A，也满足
    // 所以earlySingletonExposure=true
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    // 还是会进入到这段代码中
    if (earlySingletonExposure) {
     // 还是会通过三级缓存提前暴露一个工厂对象
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
    ```

    看到了吧，即使没有循环依赖，也会将其添加到三级缓存中，而且是不得不添加到三级缓存中，因为到目前为止Spring也不能确定这个Bean有没有跟别的Bean出现循环依赖。

    假设我们在这里直接使用二级缓存的话，那么意味着所有的Bean在这一步都要完成`AOP`代理。这样做有必要吗？

    不仅没有必要，而且违背了Spring在结合`AOP`跟Bean的生命周期的设计！Spring结合`AOP`跟Bean的生命周期本身就是通过`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来完成的，在这个后置处理的`postProcessAfterInitialization`方法中对初始化后的Bean完成`AOP`代理。如果出现了循环依赖，那没有办法，只有给Bean先创建代理，但是没有出现循环依赖的情况下，设计之初就是让Bean在生命周期的最后一步完成代理而不是在实例化后就立马完成代理。

## **三级缓存真的提高了效率了吗？**

1. 没有进行`AOP`的Bean间的循环依赖

    从上文分析可以看出，这种情况下三级缓存根本没用！所以不会存在什么提高了效率的说法

2. 进行了`AOP`的Bean间的循环依赖

    就以我们上的A、B为例，其中A被`AOP`代理，我们先分析下使用了三级缓存的情况下，A、B的创建流程

    ![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109271016330.jpg)

    假设不使用三级缓存，直接在二级缓存中

    ![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109271016364.jpg)

    上面两个流程的唯一区别在于为A对象创建代理的时机不同，在使用了三级缓存的情况下为A创建代理的时机是在B中需要注入A的时候，而不使用三级缓存的话在A实例化后就需要马上为A创建代理然后放入到二级缓存中去。对于整个A、B的创建过程而言，消耗的时间是一样的

    综上，不管是哪种情况，三级缓存提高了效率这种说法都是错误的！

    # 总结

    Spring是怎么解决循环依赖的？

    Spring通过三级缓存解决了循环依赖问题，其中以及缓存为单例池（singletonObjects）,二级缓存为早期曝光对象earlySingletonObjects，三级缓存为早期曝光对象工厂singletonFactories。当A、B两个类发生循环引用时，在A完成实例化后，就是用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。当A进行属性注入时，会去创建B，同时，B又依赖了A，所以创建B的同时又会去getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取，第一步，先获取到三级缓存中的工厂；第二步，调用对象工厂的getObject方法来获取到对应的对象，得到这个对象后将其注入到B中。紧接着B会走完它的生命周期流程。当B创建完后，会将B再注入到A中，此时A在完成它的整个生命周期。至此，循环依赖结束！

    

    为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？

    如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理。

# 参考

[面试必杀技，讲一讲Spring中的循环依赖](https://zhuanlan.zhihu.com/p/157314153))

