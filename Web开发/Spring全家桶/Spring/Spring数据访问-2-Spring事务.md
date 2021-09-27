# 数据库事务

事务（Transaction）是面向关系型数据库（RDBMS）企业应用程序的重要组成部分，用来确保数据的完整性和一致性。

事务具有以下 4 个特性，即原子性、一致性、隔离性和持久性，这 4 个属性称为 ACID 特性。

- 原子性（Atomicity）：一个事务是一个不可分割的工作单位，事务中包括的动作要么都做要么都不做。
- 一致性（Consistency）：事务必须保证数据库从一个一致性状态变到另一个一致性状态，一致性和原子性是密切相关的。
- 隔离性（Isolation）：一个事务的执行不能被其它事务干扰，即一个事务内部的操作及使用的数据对并发的其它事务是隔离的，并发执行的各个事务之间不能互相打扰。
- 持久性（Durability）：持久性也称为永久性，指一个事务一旦提交，它对数据库中数据的改变就是永久性的，后面的其它操作和故障都不应该对其有任何影响。

## 事务隔离级别

| 方法                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| ISOLATION_DEFAULT             | 使用后端数据库默认的隔离级别                                 |
| ISOLATION_READ_UNCOMMITTED    | 允许读取尚未提交的更改，可能导致脏读、幻读和不可重复读       |
| ISOLATION_READ_COMMITTED      | （Oracle 默认级别）允许读取已提交的并发事务，防止脏读，可能出现幻读和不可重复读 |
| ==ISOLATION_REPEATABLE_READ== | ==（MySQL 默认级别），多次读取相同字段的结果是一致的，防止脏读和不可重复读，可能出现幻读== |
| ISOLATION_SERIALIZABLE        | 完全服从 ACID 的隔离级别，防止脏读、不可重复读和幻读         |

## 事务传播

| 名称                      | 说明                                                       |
| ------------------------- | ---------------------------------------------------------- |
| PROPAGATION_MANDATORY     | 支持当前事务，如果不存在当前事务，则引发异常               |
| PROPAGATION_NESTED        | 如果当前事务存在，则在嵌套事务中执行                       |
| PROPAGATION_NEVER         | 不支持当前事务，如果当前事务存在，则引发异常               |
| PROPAGATION_NOT_SUPPORTED | 不支持当前事务，始终以非事务方式执行                       |
| ==PROPAGATION_REQUIRED==  | ==默认传播行为，支持当前事务，如果不存在，则创建一个新的== |
| PROPAGATION_REQUIRES_NEW  | 创建新事务，如果已经存在事务则暂停当前事务                 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果不存在事务，则以非事务方式执行           |

# Spring事务

Spring 的事务管理有 2 种方式：

1. 传统的编程式事务管理，即通过编写代码实现的事务管理；
2. 基于 AOP 技术实现的声明式事务管理。

## 编程式事务管理

通过事务管理接口实现事务管理，活性高，但难以维护。

### ^*^事务管理接口

PlatformTransactionManager、TransactionDefinition 和 TransactionStatus 是事务的 3 个核心接口。

## 声明式事务管理

Spring 声明式事务管理是使用AOP实现的，其最大的优点在于无须通过编程的方式管理事务，只需要在配置文件中进行相关的规则声明，就可以将事务规则应用到业务逻辑中。

Spring 实现声明式事务管理主要有 2 种方式：

- 基于 XML 方式的声明式事务管理。
- 通过 Annotation 注解方式的事务管理。

显然声明式事务管理要优于编程式事务管理。

### 注解实现事务管理

1. 在Spring容器中注册驱动

    ```xml
    <tx:annotation-driven transaction-manager="txManager"/>
    ```

2. 在需要使用事务的业务类或者方法中添加注解 ==**@Transactional**==，并配置 @Transactional 的参数。常用参数如下

    | 参数        | 说明                       |
    | ----------- | -------------------------- |
    | timeout     | 事务超时时间               |
    | readOnly    | 设置时读写事务还是只读事务 |
    | isolation   | 设置事务的隔离级别         |
    | propagation | 设置事务的传播行为         |



# 常见面试问题

## Spring事务管理原理

首先，我们先明白spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。
那么，我们一般使用JDBC操作事务的时候，代码如下

- (1)获取连接 Connection con = DriverManager.getConnection()
- (2)开启事务con.setAutoCommit(true/false);
- (3)执行CRUD
- (4)提交事务/回滚事务 con.commit() / con.rollback();
- (5)关闭连接 conn.close();

使用spring事务管理后，我们可以省略步骤(2)和步骤(4)，就是让AOP帮你去做这些工作。

## :small_red_triangle: spring事务什么时候失效? 

spring事务的原理是AOP，进行了切面增强，那么失效的根本原因是这个AOP不起作用了！常见情况有如下几种

### 发生自调用

![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109262236386.jpg)

此时是无效的，因为等同于下面的代码

![img](https://gitee.com/huawesome/my-picture/raw/master/img/202109262237119.jpg)

此时这个this对象不是代理类，而是UserService对象本身！
解决方法很简单，让那个this变成UserService的代理类即可

### 方法不是public的

@Transactional注解的方法都是被外部其他类调用才有效！
如果方法修饰符是private的，这个方法能被外部其他类调到么？
既然调不到，事务生效有意义么？
想通这套逻辑就行了~~

记住:@Transactional 注解只能应用到 public 可见度的方法上。如果你在 protected、private 或者 package-visible 的方法上使用 @Transactional 注解，它也不会报错， 但是这个被注解的方法将不会有事务行为。

### 发生了错误异常

默认回滚的是：RuntimeException。如果是其他异常想要回滚，需要在@Transactional注解上加rollbackFor属性。
又或者是异常被吞了，事务也会失效，不赘述！

### 数据库不支持事务

### 配置错误

- Springboot中的Application类上不加注解`@EnableTransactionManagement`

- 把隔离级别配置成

    > @Transactional(propagation = Propagation.NOT_SUPPORTED)

## Spring的事务隔离和数据库的事务隔离是一个概念么？

是，以Spring配置为准，源码里有个方法写了

## spring事务控制放在service层，在service方法中一个方法调用service中的另一个方法，默认开启几个事务？

此题考查的是spring的事务传播行为
我们都知道，默认的传播行为是PROPAGATION_REQUIRED，==如果外层有事务，则当前事务加入到外层事务，一块提交，一块回滚。如果外层没有事务，新建一个事务执行！==
也就是说，默认情况下只有一个事务！

当然这种时候如果面试官继续追问其他传播行为的情形，如何回答？

那我们应该？我们应该？把每种传播机制都拿出来讲一遍？没必要，这种时候直接掀桌子走人。因为你就算背下来了，过几天还是忘记。用到的时候，再去查询即可。

## 怎么保证spring事务内的连接唯一性？

这道题很多种问法，例如Spring 是如何保证事务获取同一个Connection的?

OK,开始我们的讲解！其实答案只有一句话，因为那个Connection在事务开始时封装在了ThreadLocal里，后面事务执行过程中，都是从ThreadLocal中取的，肯定能保证唯一，因为都是在一个线程中执行的!

## 参考

[参考](https://zhuanlan.zhihu.com/p/151351423)







