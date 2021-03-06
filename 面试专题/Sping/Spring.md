# Spring基础

1. 为什么会有Spring，Spring解决了什么问题？

[Spring-核心思想：Bean的构建、自动注入和AOP](https://app.gitbook.com/@1184884206/s/java/web-kai-fa/spring-quan-jia-tong/spring/spring-he-xin-si-xiang-bean-de-gou-jian-zi-dong-zhu-ru-he-aop)

2. IOC容器是什么？如何实现一个IOC容器？

[Spring核心原理-IOC](https://app.gitbook.com/@1184884206/s/java/web-kai-fa/spring-quan-jia-tong/spring/spring-he-xin-yuan-li-1ioc)

3. 详细说说Spring是如何扫描得到Bean的？

    [本地链接](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-1-IOC.md#简单实现一个IOC（直接使用的bean的来源）)

4. ApplicationContext和BeanFactory有什么区别？

    [本地链接](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-3-Bean.md#BeanFactory和ApplicationContext)

5. 如何实现AOP？项目什么地方实现了AOP？

    [本地链接](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-2-AOP.md#Spring核心原理-2-AOP-AOP基础)

6. Spring Bean的自动装配？有哪些方式？

    [本地链接](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-3-Bean.md#:warning: 自动装配)

7. Spring容器启动流程是怎样的？

    3. [本地链接](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-1-IOC.md#简单实现一个IOC（直接使用的bean的来源）)

8. [Spring中的Bean是线程安全的吗？](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-3-Bean.md#Spring Bean的线程安全问题)

9. [Spring中的事务是什么实现的？](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring数据访问-2-Spring事务.md)

10. [Spring事务什么时候会失效？](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring数据访问-2-Spring事务.md)

11. [请讲一讲Spring中的循环依赖？](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring源码-1-Spring循环依赖.md)

12. 单例模式在Spring框架中的应用？

13. Spring-Spring用到了哪些设计模式？

14. Spring-@Configuration的作用和底层原理？

     @Configuration与@Bean都是来自spring的注解，作用是使用类来代替xml配置文件的功能。

     @Configuration

     @configuration用在类上方，声明这个类是一个spring配置类，**执行的功能是代替spring的配置文件**

     也可以说，相当与sprig.xml中的<beans>标签

     ```java
     @Configuration
     public class ConfigurationTest {
         public ConfigurationTest(){
             System.out.println("this is spring.xml");
             System.out.println("this is @configuration");
         }
     }
     ```

     [参考](https://zhuanlan.zhihu.com/p/335068145)



















