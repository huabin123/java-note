### 带着面试题去理解

- 什么是ThreadLocal? 用来解决什么问题的?

  > 比如说数据库连接管理在多线程中不使用同步的话会出现线程安全问题，比如多次创建connect，或者在调用connect的地方不能做到同步保障安全。使用同步每次都在方法内部创建连接可以解决这个问题，但是会导致服务器压力非常大，并且严重影响程序执行性能。所以需要一种能在各个线程内部使用线程自有变量的技术，能在线程内部任何地方使用，线程之间互不影响，这就是ThreadLocal出现的原因

- 说说你对ThreadLocal的理解

- ThreadLocal是如何实现线程隔离的?

- 为什么ThreadLocal会造成内存泄露? 如何解决

- 还有哪些使用ThreadLocal的应用场景?



### ThreadLocal简介

ThreadLocal顾名思义thread线程local本地，是线程本地的一个东西。也就是说`变量是线程本地`的，在线程之间是隔离的。一般用来保存要重复使用的数据，并且每个线程都需要使用自己独有的，例如用户信息，传一些平台信息，版本号的时候会使用。一般是在最开始用作过滤器的核心部分。

需要注意的是在每一次读取完ThreadLocal中的值之后最好使用remove()清理掉，这是为了当这个线程被重新使用的时候产生数据干扰

### ThreadLocal原理？

ThreadLocal本身不保存数据，ThreadLocal类中有一个map，用来存储每一个线程中的变量。



### ThreadLocal使用场景

> session管理和数据库链接管理

**参考**

https://www.bilibili.com/video/BV1SJ41157oF?from=search&seid=7117755280068542819