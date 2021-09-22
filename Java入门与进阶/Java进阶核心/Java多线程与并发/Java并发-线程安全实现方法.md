# 线程安全实现方法

## 互斥同步

### synchonized

### ReentranLock

## 非阻塞同步

### CAS

#### 概念

CAS的全称是Compare-And-Swap（比较和交换），它是CPU并发原语

它的功能是判断内存某个位置的值是否为预期值，如果是则更新为新的值，这个过程是原子的。

它是一种无锁算法，即在不使用锁的情况下对多线程之间的变量进行同步、CAS算法涉及到三个操作数：

1. 需要读写的内存值V
2. 进行比较的值A
3. 拟写入的新值B

当且仅当V==A时，才会通过CAS的原子方式用B更新V，否则不会进行任何操作（比较和替换是一个原子操作）。一般情况下是一个自旋操作，即不断地重试。

```java
public class CASDemo {

    public static void main(String[] args) {
        // 创建一个原子类
        AtomicInteger atomicInteger = new AtomicInteger(5);

        /*compareAndSet第一个参数是期望值，第二个是更新值，只有期望值和原来的值相等时，才会更新*/
        System.out.println(atomicInteger.compareAndSet(5,2021)+"\t current data: "+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5,2020)+"\t current data: "+atomicInteger.get());
    }

}
```

```sh
true	 current data: 2021
false	 current data: 2021
```



#### CAS底层原理

首先看atomicInteger.getAndIncrement()方法的源码，可以看出来，底层调用了一个unsafe类的getAndAddInt方法

1. **unsafe类**

   ![image-20210922071407868](https://gitee.com/huawesome/my-picture/raw/master/img/202109220714910.png)

   Unsage是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（Native）方法来访问，Unsafe相当于一个后门，基于该类可以操作特定的内存数据。Unsafe类存在于sun.misc包中，其内部方法可以向C的指针一样直接操作内存，因为Java中的CAS操作的执行依赖于Unsafe类的方法。

   注意Unsafe类的所有方法都是native修饰的，也就是说unsafe类中的方法都直接调用操作系统的底层资源执行相应的任务

   ```java
   public native int getIntVolatile(Object var1, long var2);
   ```

   为什么Atomic修饰的包装类能够保证原子性，依靠的就是底层的unsafe类

2. **变量valueOffset**

   表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。

   ![image-20210922072655557](https://gitee.com/huawesome/my-picture/raw/master/img/202109220726592.png)

   从这里可以看到，通过valueOffset，直接通过内存地址，获取到值，然后进行加一操作

   - [ ] `Todo`:dagger: 变量valueOffset

3. 变量value用volatile修饰

   保证了多线程之间的内存可见性

   ![在这里插入图片描述](https://gitee.com/huawesome/my-picture/raw/master/img/202109220832213.png)

   - Var5：用var1和var2找到内存中的真实值
   - 用AtomicInteger对象本身的值与var5比较
   - 如果相同，更新var5+var4并返回true
   - 如果不同，继续取值然后再比较，直到更新完成

   操作的时候，需要比较工作内存的值和主内存的值，假设compareAndSwapInt一直返回false，那么while方法就会一直循环下去，直到期望的值和真实值一样。

   假设线程A和线程B同时执行getAndInt操作（分别跑在不同的CPU上）

   - AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的 value 为3，根据JMM模型，线程A和线程B各自持有一份价值为3的副本，分别存储在各自的工作内存
   - 线程A通过getIntVolatile(var1 , var2) 拿到value值3，这是线程A被挂起（该线程失去CPU执行权）
   - 线程B也通过getIntVolatile(var1, var2)方法获取到value值也是3，此时刚好线程B没有被挂起，并执行了compareAndSwapInt方法，比较内存的值也是3，成功修改内存值为4，线程B打完收工，一切OK
   - 这时线程A恢复，执行CAS方法，比较发现自己手里的数字3和主内存中的数字4不一致，说明该值已经被其它线程抢先一步修改过了，那么A线程本次修改失败，只能够重新读取后在来一遍了，也就是在执行do while
   - 线程A重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功。

> Unsafe类 + CAS思想： 也就是自旋，自我旋转

#### CAS缺点

CAS不加锁，保证一致性，但是需要多次比较

- 循环时间长，开销大，如果不成功则会一致循环，最差的情况，就是某个线程一直取到的值与预期值都不一样，这样会无限循环；
- 只能保证一个共享变量的原子操作
- 引出ABA问题

#### ABA问题

CAS算法实现的一个重要前提，需要取出内存某一时刻的数据，并在当下时刻比较并替换，这个时间差会导致数据的变化。就是在中间的时候有别的线程对该值进行了更新，但是又在此线程获取该变量的值的时候又改回去了。尽管CAS操作成功，但是不代表过程就是没问题的。

#### 解决ABA问题

新增一种机制，也就是修改版本号。

当线程A更新数据时，还会读取version值。在提交更新时，如果数据库中的version值等于读取出来的version值才会更新。否则提交失败，需要重试知道更新成功。

应用：AtomicStampedReference



#### 参考

https://blog.csdn.net/weixin_45007916/article/details/108080745

### Atomic类

## 无同步方案

### 栈封闭

### 线程本地存储（ThreadLocal）

### 可重入代码（Reentrant Code）
