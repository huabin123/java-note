# Spring基础

1. 为什么会有Spring，Spring解决了什么问题？

[Spring-核心思想：Bean的构建、自动注入和AOP](https://app.gitbook.com/@1184884206/s/java/web-kai-fa/spring-quan-jia-tong/spring/spring-he-xin-si-xiang-bean-de-gou-jian-zi-dong-zhu-ru-he-aop)

2. IOC容器是什么？如何实现一个IOC容器？

[Spring核心原理-IOC](https://app.gitbook.com/@1184884206/s/java/web-kai-fa/spring-quan-jia-tong/spring/spring-he-xin-yuan-li-1ioc)

3. 详细说说Spring是如何扫描得到Bean的？

4. ApplicationContext和BeanFactory有什么区别？

- ApplicationContext是BeanFactory的子接口；
- BeanFactory是一个底层的IOC容器，提供了IOC容器的基本实现，而ApplicationContext则是BeanFactory的超集提供了丰富的企业级特性；
- ApplicationContext是委托DefaultListableBeanFactory来实现Bean的依赖查找和依赖注入。

5. 如何实现AOP？项目什么地方实现了AOP？

    [本地链接](/Users/apple/Workspace/java-note/Web开发/Spring全家桶/Spring/Spring核心原理-2-AOP.md#Spring核心原理-2-AOP-AOP基础)

    



