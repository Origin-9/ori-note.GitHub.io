## AQS介绍

AQS全称为AbstractQueuedSynchronizer，在java.util.concurrent.locks下面

AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，如我们提到的ReentrantLock，Semaphore，ReentrantReadWriteLock

## AQS原理分析

### 原理总览

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

![image-20200830172228464](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200830172228464.png)

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

### AQS对资源的共享方式

- 独占：只有一个线程能拿到锁，如ReentrantLock，公平锁和非公平锁
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- 共享：多个线程可同时执行，如Semaphore/CountDownLatch

ReentrantReadWriteLock 可以看成是组合式，读锁是共享，写锁是独占

### AQS底层使用模板方法模式

如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

同步器还提供了以下常用的模板方法：

> - acquire(int arg)：独占式获取同步状态，内部是通过 `tryAcquire(int arg)` 方法实现的。
> - acquireInterruptibly(int arg)：与 `acquire(int arg)` 相同，但是该方法可以响应中断。
> - tryAcquireNanos(int arg, long nanosTimeout)：在 `acquireInterruptibly(int arg)` 方法的基础上增加了超时功能。
> - acquireShared(int arg)：共享式获取同步状态，内部是通过 `tryAcquireShared(int arg)` 方法实现的。
> - acquireSharedInterruptibly(int arg)：与 `acquireShared(int arg)` 方法相同，但是该方法可以响应中断。
> - tryAcquireSharedNanos(int arg, long nanosTimeout)：在 `acquireSharedInterruptibly(int arg)` 方法的基础上增加了超时功能。
> - release(int arg)：独占式释放同步状态，内部是通过 `tryRelease(int arg)` 方法实现的。
> - releaseShared(int arg)：共享式释放同步状态，内部是通过 `tryReleaseShared(int arg)` 方法实现的。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

###  AQS 组件总结

- **Semaphore(信号量)-允许多个线程同时访问：** synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。

  Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

- **CountDownLatch （倒计时器）：** CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。

  维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

- **CyclicBarrier(循环栅栏)：** CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await()方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。

  和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

### AQS设计妙处

#### 1. 自旋锁

死循环+CAS实现

例如在通过自旋尝试获取锁，获取失败进入阻塞（被唤醒后，从阻塞处继续自旋）

自旋操作把当前节点加入队列中，

#### 2. 模板方法

