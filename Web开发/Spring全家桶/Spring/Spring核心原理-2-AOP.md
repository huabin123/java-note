---
title: Spring核心原理-AOP
author: huabin
vlook-query: effects=2&ws=none&lmc=3
vlook-doc-lib: 
---

[TOC]

# Spring核心原理-2-AOP-AOP基础

## 基础概念

### AOP

AOP（Aspect Oriented Programming），`面向切面编程`，==不影响原有功能的情况下达到动态方法增强的效果。==

Spring AOP则利用`CGlib`和`JDK动态代理`等方式来实现运行。

==AOP使用场景==：日志、监控、性能统计、异常处理、权限、统一认证等。

总而言之，我们之所以能无感知的在容器对象方法前后任意添加代码片段，那是由于Spring在运行期帮我们把切面中的代码逻辑动态“织入”到了容器对象方法内，所以所`AOP本质上就是一个代理模式。`

### AOP和OOP的区别

AOP是OOP的补充，当我们需要为多个对象引入一个公共行为，比如日志，操作记录等，就需要在每个对象中引用公共行为，这样程序就产生了大量的重复代码，使用AOP可以完美解决这个问题。

### AOP核心概念

###### AOP术语

| 名称               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| Joinpoint(连接点)  | 指那些被拦截到的点，在Spring中，只可以被动态代理拦截目标类的方法。 |
| Pointcut（切入点） | 指要对哪些Joinpoint进行拦截，即被拦截的连接点。              |
| Advice（通知）     |                                                              |
| Target（目标）     |                                                              |
| Weaving（植入）    |                                                              |
| Proxy（代理）      |                                                              |
| Aspect（切面）     |                                                              |

###### Advice（通知）的五种类型

| 通知            | 说明 |
| --------------- | ---- |
| before          |      |
| after           |      |
| after-returning |      |
| After-throwing  |      |
| around          |      |

## Spring实现AOP的两种方式

### JDK动态代理

JDK代理是用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

Spring JDK动态代理需要实现InvocationHandler接口，重写invoke方法，客户端使用Java.lang.reflect.Proxy类产生动态代理的对象。

[tiaozhuan](#示例)

#### 实现思路

1. 定义自定义切面类，定义通知类型，myBefore(),myAfter();
2. 定义JDK动态代理实现类实现InvocationHandler接口
    1. 定义需要代理的目标对象
    2. 实例化自定义切面类
    3. 重写invoke方法，定义切面中通知的执行顺序
3. 定义获取代理对象的方法，参数为实现类对象，使用Proxy.newProxyInstance获取，==只能代理实现了接口的类==
4. 执行类
    1. 实例化JDKProxy对象
    2. 获取代理对象
    3. 执行相应方法

#### 示例

![JDK动态代理实现AOP]

```java
```



### CGLIB动态代理

CGLIB是一个高性能开源的代码生成包，Spring核心包集成了CGLIB。

CGLIB动态代理需要实现MethodInterceptor接口，重写intercept拦截方法，加载对象类的class文件，通过修改其字节码生成子类来处理。

#### 实现思路

1. 定义自定义切面类，定义通知类型，myBefore(),myAfter();
2. 定义CGLIB动态代理类，实现MethodInterceptor接口
    1. 重写拦截方法，定义通知（自定义切面类的方法）
    2. 定义获取代理对象方法
        1. 为目标对象赋值
        2. 设置父类（CGLIB是针对制定对象生成一个子类）
        3. 设置回调
        4. 创建并返回代理对象
3. 执行
    1. 实例化CGLIBProxy对象
    2. 获取代理对象
    3. 执行对应方法

#### CGLIB示例

![JDK动态代理实现AOP]

```java

```



### JDK代理和CGLIB代理的区别

实现方式的不同

- JDK动态代理只能对实现了接口的类生成代理，而不能针对类。

- CGLIB是针对类实现代理，主要是对制定的类生成一个子类，覆盖其中的方法。因为是继承，所以该类或方法不能声明成final类型

性能比较：JDK>CGLIB

## AspectJ-AOP的具体实现

### AspectJ

AspectJ 是一个基于 Java 语言的 AOP 框架，它扩展了 Java 语言，提供了强大的 AOP 功能。

使用 AspectJ 需要导入以下 jar 包：

- Aspectjrt.jar
- Aspectjweaver.jar
- Aspectj.jar

[jar 包下载地址](https://www.eclipse.org/aspectj/downloads.php)

使用 AspectJ 开发 AOP 通常有以下 2 种方式：

- 基于 XML 的声明式 AspectJ
- 基于 Annotation 的声明式

### AspectJ基于XML开发AOP

基于XML的声明式是指通过Spring配置文件的方式来定义切面、切入点及通知，而所有的切面和通知都必须定义在`<aop:config>`元素中。

#### 实现步骤

1. 导入Spring AOP命名空间
2. 定义切面`<aop:aspect>`
3. 定义切入点`<aop:pointcut>`
4. 定义通知

#### 示例

![AspectJ基于XML开发AOP]

```java

```



### AspectJ基于注解开发AOP

在Spring中，尽管使用XML配置文件可以实现AOP开发，但是如果所有的相关配置都放在配置文件中，势必会导致XML配置文件过于臃肿，从而给维护和升级带来一定的困难。

为此，AspectJ 框架为 AOP 开发提供了一套注解。AspectJ 允许使用注解定义切面、切入点和增强处理，Spring 框架可以根据这些注解生成 AOP 代理。

![Annotation注解介绍]

| 名称            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| @Aspect         | 用于定义一个切面。                                           |
| @Pointcut       | 用于定义一个切入点。                                         |
| @Before         | 用于定义前置通知，相当于 BeforeAdvice。                      |
| @AfterReturning | 用于定义后置通知，相当于 AfterReturningAdvice。              |
| @Around         | 用于定义环绕通知，相当于MethodInterceptor。                  |
| @AfterThrowing  | 用于定义抛出通知，相当于ThrowAdvice。                        |
| @After          | 用于定义最终final通知，不管是否异常，该通知都会执行。        |
| @DeclareParents | 用于定义引介通知，相当于IntroductionInterceptor（不要求掌握）。 |

#### 启用方式

1. 使用@Configuration和@EnableAspectJAutoProxy
2. 基于XML配置

#### 实现步骤

1. 定义切面@Aspect
2. 定义切入点@Pointcut
3. 定义通知advice

#### 示例

## 参考

[C语言编程网：Spring AOP](http://c.biancheng.net/spring/aspectj-xml.html)















































