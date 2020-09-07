## 使用线程

有三种使用线程的方法：

- 实现 Runnable 接口；

- 实现 Callable 接口；

  需要借助RunnableFuture实现类，传入Thread构造方法中。

- 继承 Thread 类。

**实现 Runnable 和 Callable 接口**的类只能当做一个可以**在线程中运行的任务**，不是真正意义上的线程，因此**最后还需要通过 Thread 来调用**。可以理解为**任务是通过线程驱动从而执行的**。

### 线程返回值

- join() 方法

  将当前线程挂起，等待调用 join() 方法线程执行完成

- FutureTask

  传入Callable实现类，调用isdone()，get() 方法

- 线程池+Callable+Future

  线程池submit方法传入Callable，返回Runnable实现类，然后调用get方法

## 线程池

### 为什么使用线程池

- 降低资源消耗。重复利用已创建的线程，减少线程创建销毁消耗
- 提高效应速度。任务到达，不用等待创建线程成功在执行
- 提高线程可管理性。线程的无限创建，不仅消耗系统资源，并且降低系统稳定性。使用线程池可以实现线程的管理，监控，调优等。

### 如何创建线程池

#### 1. 构造方法实现

```java
ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize` :** 核心线程数线程数定义了最小可以同时运行的线程数量。
- **`maximumPoolSize` :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **`workQueue`:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

`ThreadPoolExecutor`其他常见参数:

1. **`keepAliveTime`**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. **`unit`** : `keepAliveTime` 参数的时间单位。
3. **`threadFactory`** :executor 创建新线程的时候会用到。可以自定义ThreadFactory（实现ThreadFactory）
4. **`handler`** :饱和策略。关于饱和策略下面单独介绍一下。

##### 饱和策略

前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任时

- **`ThreadPoolExecutor.AbortPolicy`**：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：调用执行自己的线程运行任务，也就是**直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务**。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
- **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。

#### 2. 工具类 Executors 创建

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；队列使用LinkedBlockingQueue
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。队列使用LinkedBlockingQueue

### execute()方法和submit()方法

1. **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
2. **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### 线程池原理

![image-20200828160815377](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200828160815377.png)

## 线程状态

- 新建（NEW）

- 可运行（RUNABLE）

  正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度。

- 阻塞（BLOCKED）

  阻塞状态

- 无限期等待（WAITING）

  等待其它线程显式地唤醒。

  ![image-20200828155621833](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200828155621833.png)

- 限期等待（TIMED_WAITING）

  无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

  ![image-20200828155634335](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200828155634335.png)

- 死亡（TERMINATED）

  线程结束任务之后自己结束，或者产生了异常而结束

## 线程之间的协作

### join()

**在线程中调用另一个线程的 join() 方法，会将当前线程挂起**，而不是忙等待，**直到目标线程结束。**

### wait()  notify() notifyAll()

调用 wait() 使得线程等待，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程

**wait() ，线程会释放锁，必须在同步方法或者同步块中使用。**

**wait() 和 sleep() 的区别**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

### await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调。

调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

## 中断

调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

调用 interrupt() 方法会设置线程的中断标记

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

## synchronized 关键字

> Java 早期版本中，synchronized属于**重量级锁**，效率低下，因为监视器锁（monitor）是**依赖于底层的操作系统的 Mutex Lock 来实现的如果要挂起或者唤醒一个线程**，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。

- **修饰实例方法:** 作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁
- **修饰静态方法:** 也就是给当前类加锁，会作用于类的所有对象实例。
- **修饰代码块:** 指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

### sychronized原理

1. 同步语句块

   **synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。**执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

2. 修饰方法

   取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

### 锁优化

JDK1.6 中synchronized的实现进行了各种优化，如适应性自旋、**锁消除、锁粗化、轻量级锁和偏向锁**，主要解决三种场景:

- 只有一个线程进入临界区，偏向锁
- 多线程未竞争，轻量级锁
- 多线程竞争，重量级锁

偏向锁→轻量级锁→（自旋锁）→重量级锁过程，**锁可以升级但不能降级**，这种策略是为了提高获得锁和释放锁的效率

![image-20200828184124031](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200828184124031.png)

#### 偏向锁

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束

#### 轻量级锁

轻量级锁是相对于传统的重量级锁而言，它**使用 CAS 操作来避免重量级锁使用互斥量的开销**。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

**当尝试获取一个锁对象时，如果锁对象标记为 0 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。**

如果 CAS 操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程线程抢占了。

如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

#### 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。**自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。**

> 在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

#### 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

#### 锁粗化

如果**一系列的连续操作都对同一个对象反复加锁和解锁**，频繁的加锁操作就**会导致性能损耗**

假如多次StringBuffer的连续append操作，就会把锁粗化到整个操作序列的外部。