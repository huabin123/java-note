# 线程基础

## 线程状态转换

状态转换：

![image-20210916172724349](https://gitee.com/huawesome/my-picture/raw/master/img/202109161727423.png)

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

**