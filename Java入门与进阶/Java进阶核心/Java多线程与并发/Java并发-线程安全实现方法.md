### 线程安全实现方法

- 互斥同步

	- synchonized
	- ReentranLock

- 非阻塞同步

	- CAS
	- Atomic类

- 无同步方案

	- 栈封闭
	- 线程本地存储（ThreadLocal）
	- 可重入代码（Reentrant Code）
