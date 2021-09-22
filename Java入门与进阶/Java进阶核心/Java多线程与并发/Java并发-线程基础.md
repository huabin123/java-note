# 线程基础

## 线程状态转换

状态转换：

![image-20210916194154926](https://gitee.com/huawesome/my-picture/raw/master/img/202109161941966.png)

生命周期：

新建（New）---可运行（Runnable）---阻塞（Blocking）---无限期等待（Waitting）---限期等待(Timed Waiting)---死亡(Terminated)

### 新建（New）

### 可运行（Runnable）

### 阻塞（Blocking）

### 无限期等待（Waiting）

### 限期等待（Timed Waiting）

### 死亡（Terminated）

## 线程使用方式

### 实现Runnable接口

### 实现Callable接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

### 继承Thread类

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

## 基础线程机制

### Executor

### Daemon

### sleep()

#### sleep()和wait()的区别

- 对于锁资源的处理不同：

  - sleep不会释放当前所占有的锁
  - wait会释放对象的锁

- 所属类的不同：

  - sleep方法是定义在Thread上
  - wait方法定义在Object上

- 使用范围的不同：

  wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用(使用范围)

- 异常的处理

  sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

### yield()

#### join()和yield()的区别

- 在很多应用场景中存在这样一种情况，主线程创建并启动子线程后，如果子线程要进行很耗时的计算，那么主线程将比子线程先结束，但是主线程需要子线程的计算的结果来进行自己下一步的计算，这时主线程就需要等待子线程，java中提供可join()方法解决这个问题。
- 对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

## 线程中断

### interruptedException

### interrupted()

### Executor的中断操作

## 线程互斥同步

### synchonized

### ReentranLock

## 线程间通信

#### 为什么要线程通信

> 当多个线程操作共享的资源时，互相告知自己的状态以避免资源争夺

#### 线程通信的方式

1. 共享内存

   `volatile`

2. 消息传递

   1. wait/notify等待通知方式
   2. join方式

3. 管道流

### join()

### wait() notify() notifyAll()

### await() signal() signalAll()

## 线程异常

### 线程异常没有被捕获的原因

Thread类中有一个dispatchUncaughtException的方法来分发未捕获的异常。

这个方法只被JVM调用

- **Thread#getUncaughtExceptionHandler**：获取UncaughtExceptionHandler接口实现类

![image-20210922100933350](https://gitee.com/huawesome/my-picture/raw/master/img/202109221009424.png)

> UncaughtExceptionHandler是Thread中定义的接口，在Thread类中，uncaughtExceptionHandler默认是null，因此该方法返回group，即实现了UncaughtExceptionHandler接口的ThreadGroup类

- **UncaughtExceptionHandler#uncaughtException：** ThreadGroup 类的 uncaughtException 方法实现

![image-20210922101400913](https://gitee.com/huawesome/my-picture/raw/master/img/202109221014945.png)

> 因为在Thread类中没有对ThreadGroup parent和Thread.getDefaultUncaughtExceptionHandler()进行赋值，因此将进入最后一层条件，将异常直接打印到控制台中，对异常不做任何处理。



### 线程异常处理

分析源码后有两个思路：

1. 让`ThreadGroup`不为null；
2. 让`UncaughtExceptionHandler`类型的变量不为null。

### 线程池异常处理

一般应用中线程都是线程池创建复用的，因此对线程池的异常处理就是为线程池工厂类`ThreadFactory`类生成的线程生成的线程添加异常处理器

- 默认异常处理器

  ```java
  Thread.setDefaultUncaughtExceptionHandler(new  ExceptionHandler());
   ExecutorService es = Executors.newCachedThreadPool();
  es.execute(new Task(i++))
  ```

- 自定义异常处理器

  ```java
  ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(1, 1,
          0L, TimeUnit.MILLISECONDS,
          new LinkedBlockingQueue<Runnable>());
  threadPoolExecutor.setThreadFactory(new MyThreadFactory());
  threadPoolExecutor.execute(new Task(i++));
  ```

- 自定义工厂类：MyThreadFactory.java

  ```java
  private static class MyThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
      Thread t = new Thread();
      //自定义UncaughtExceptionHandler
      t.setUncaughtExceptionHandler(new ExceptionHandler());
      return t;
     }
  }
  ```

  > 为什么异常要由线程自身进行捕获？

  来自JVM的设计理念，“线程是独立执行的代码片段，线程的问题应该由线程自己来解决，而不要委托到外部”。因此，任务抛出的异常应该在线程代码边界之内处理掉，而不应该在线程方法外面由其他线程处理。

### 线程执行 Callable 型任务时的异常处理

Callable 任务抛出的异常能在代码中通过 try-catch 捕获到，但是**只有调用 get 方法**后才能捕获到