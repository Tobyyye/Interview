https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

用ReentrantLock 理解 AQS

https://blog.csdn.net/qq_31457665/article/details/110187854

概述



- 了解 AQS 么？讲讲底层实现原理 

本文主角正式登场。

AQS，全名AbstractQueuedSynchronizer，是一个抽象类的队列式同步器，它的内部通过维护一个状态volatile int state(共享资源)，一个FIFO线程等待队列来实现同步功能。

state用关键字volatile修饰，代表着该共享资源的状态一更改就能被所有线程可见，而AQS的加锁方式本质上就是多个线程在竞争state，当state为0时代表线程可以竞争锁，不为0时代表当前对象锁已经被占有，其他线程来加锁时则会失败，加锁失败的线程会被放入一个FIFO的等待队列中，这些线程会被`UNSAFE.park()`操作挂起，等待其他获取锁的线程释放锁才能够被唤醒。



![img](https://img-blog.csdnimg.cn/20201126150417950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNDU3NjY1,size_16,color_FFFFFF,t_70)





### 基础定义

AQS支持两种资源分享的方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

自定义的同步器继承AQS后，只需要实现共享资源state的获取和释放方式即可，其他如线程队列的维护（如获取资源失败入队/唤醒出队等）等操作，AQS在顶层已经实现了，

AQS代码内部提供了一系列操作锁和线程队列的方法，主要操作锁的方法包含以下几个：

- compareAndSetState()：利用CAS的操作来设置state的值
- tryAcquire(int)：独占方式获取锁。成功则返回true，失败则返回false。
- tryRelease(int)：独占方式释放锁。成功则返回true，失败则返回false。
- tryAcquireShared(int)：共享方式释放锁。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式释放锁。如果释放后允许唤醒后续等待结点返回true，否则返回false。

像ReentrantLock就是实现了自定义的tryAcquire-tryRelease，从而操作state的值来实现同步效果。

除此之外，AQS内部还定义了一个静态类Node，表示CLH队列的每一个结点，该结点的作用是对每一个等待获取资源做了封装，包含了需要同步的线程本身、线程等待状态.....

- AQS 有那些实现？ 

  

### 3.2 JUC中的应用场景

除了上边ReentrantLock的可重入性的应用，AQS作为并发编程的框架，为很多其他同步工具提供了良好的解决方案。下面列出了JUC中的几种同步工具，大体介绍一下AQS的应用场景：

| 同步工具               | 同步工具与AQS的关联                                          |
| :--------------------- | :----------------------------------------------------------- |
| ReentrantLock          | 使用AQS保存锁重复持有的次数。当一个线程获取锁时，ReentrantLock记录当前获得锁的线程标识，用于检测是否重复获取，以及错误线程试图解锁操作时异常情况的处理。 |
| Semaphore              | 使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。 |
| CountDownLatch         | 使用AQS同步状态来表示计数。计数为0时，所有的Acquire操作（CountDownLatch的await方法）才可以通过。 |
| ReentrantReadWriteLock | 使用AQS同步状态中的16位保存写锁持有的次数，剩下的16位用于保存读锁的持有次数。 |
| ThreadPoolExecutor     | Worker利用AQS同步状态实现对独占线程变量的设置（tryAcquire和tryRelease）。 |