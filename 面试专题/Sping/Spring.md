### 为什么会有Spring，Spring解决了什么问题？

[Spring-核心思想：Bean的构建、自动注入和AOP](https://app.gitbook.com/@1184884206/s/java/web-kai-fa/spring-quan-jia-tong/spring/spring-he-xin-si-xiang-bean-de-gou-jian-zi-dong-zhu-ru-he-aop)



### ApplicationContext和BeanFactory有什么区别？

- ApplicationContext是BeanFactory的子接口；
- BeanFactory是一个底层的IOC容器，提供了IOC容器的基本实现，而ApplicationContext则是BeanFactory的超集提供了丰富的企业级特性；
- ApplicationContext是委托DefaultListableBeanFactory来实现Bean的依赖查找和依赖注入。