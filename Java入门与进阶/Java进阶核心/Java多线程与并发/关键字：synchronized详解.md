### Synchronized有什么用？和Reetrantlock的区别？

- 共同点
  - 都是用来协调多线程对共享对象、变量的访问
  - 都是可重入锁，同一线程可以多次获得同一个锁
  - 都保证了`可见性`和`互斥性`
- 不同点
  - ReentranLock显式的获得锁，Synchronized隐式获得释放锁
  - ReentranLock是API级别的，Synchronized是JVM级别的
  - 底层实现不一样，Synchonized是同步阻塞，使用的是`悲观并发策略`，lock是同步非阻塞，采用的是`乐观并发策略`
  - synchronized在发生异常时，会自动释放线程占有的锁，因此`不会导致死锁现象发生`；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则`很可能造成死锁现`，因此使用Lock时需要在finally块中释放锁。
  - Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断。

