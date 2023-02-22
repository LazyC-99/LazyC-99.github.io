---
title: JUC并发编程
date: 2022-05-08 10:52:44
tags: [java]
---

# 创建多线程

1. 继承Thread类	重写run()方法	启动: `new MyThread().start()`

2. 实现Runnable接口	重写run()方法 	启动: `new Thread(new MyThread()).start()`

3. 实现Callable接口 	重写call()方法	启动: `new Thread( new FutureTask( new MyThread() ) )`

4. 使用线程池 new ThreadPoolExecutor() ( 主要使用 )

    **7大核心参数**

    - corePoolSize:核心池大小
    - maximumPoolSize:最大线程数
    - keepAliveTime:线程没有任务时最多保持的时间
    - keepAliveTime:超时时间单位
    - BlockingQueue<Runnable>: 阻塞队列
    - ThreadFactory: 线程工厂,创建线程
    - RejectedExecutionHandler: 拒绝策略 4种
    
    **具体使用**
    
    - 一池N线程: Executors.newFixedThreadPool()
    
    - 一池一线程: Executors.newSingleThreadExecutor()
    
    - 根据需求扩容: Executors.newCachedThreadPool()
    
    **拒绝策略(超过最大线程和阻塞队列)：**
    
    - (默认)ThreadPoolExecutor.AbortPolicy(): 直接抛出RejectedExecutionException异常阻止系统正常运行
    
    - ThreadPoolExecutor.CallerRunsPolicy(): "调用者运行“一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量.
    
    - ThreadPoolExecutor.DiscardOldestPolicy(): 抛弃队列中等待最久的任务，然后把当前任务加人队列中尝试再次提交当前任务。
    
    - ThreadPoolExecutor.DiscardPolicy(): 该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。

    **自定义线程池:** 

    ​	阿里巴巴开发手册强制线程池不允许使用Executors去创建

     * FixedThreadPool 和 SingleThreadPool:

       允许请求队列长度为Integer.MAX_VALUE,可能堆积大量的请求

    * CachedThreadPool和 ScheduledThreadPool

      允许创建线程数量为Integer.MAX_VALUE,可能堆积大量的线程

## CompletableFuture

- FutureTask缺点
  - get()方法在Future计算完成之前会一直阻塞
  - 轮询方式isDone(), 消耗cpu资源
  
- CompletableFuture优化
  - CompletableFuture提供了一种观察者模式类似的机制，可以让任务执待完成后通知监听的一方。
  
  - CompletableFuture.runAsync(<Runnable>): 无返回值
  
  - CompletableFuture.supplyAsync(<Runnable>): 有返回值
  
  - whenComplete(): 完成后的感知
  
  - exceptionally():
  
  - handle(): 完成后的处理
  
    
  
  - thenCombie(): 组合两个future,获取两个future的返回结果，有返回值
  
  - thenAcceptBoth(): 组合两个future，获取两个future任务的返回结果，然后处理任务，没有返回值。
  
  - runAfterBoth(): 组合两个future，不获取future的结果，两个future处理完任务后，处理该任务。
  
    
  
  - applyIToEither:两个任务有一个执行完成，获取返回值，处理任务并有新的返回值。
  
  - acceptEither:两个任务有一个执行完成，获取返回值，处理任务，没有新的返回值。
  
  - runAfterEither:两个任务有一个执行完成，不获取future的结果，处理任务，也没有返回值。
  
    
  
  - allof(): 组合多任务

## 常用方法

Thread类中:

- start()∶启动当前线程;调用当前线程的run()

* run()∶通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中

- yield()∶释放当前cpu的执行权,礼让线程，暂停线程 直接进入就绪状态 不是阻塞状态

- join():在线程中调用线程b的join(),此时线程a就进入阻塞状态，直到线程a完全执行完以后，线程a结束阻塞状态。
- sleep(long millitime):让当前线程睡眠指定的millitime毫秒, 当前线程是阻塞状态。

- stop(): 已过时。当执行此方法时，强制结束当前线程。

* setPriority(int p): 设置线程优先级
* setDaemon(true): 把一个用户线程编程守护线程
* getName()∶获取当前线程的名字
* currentThread()∶静态方法，返回执行当前代码的线程
* isAlive()∶判断当前线程是否存活

## 生命周期

- 新建: 当一个Thread类或其子类的对象被声明并创建时，新生的线程对象处于新建状态
- 就绪: 处于新建状态的线程被start()后，将进入线程队列等待CPU时间片，此时它已具备了运行的条件，只是没分配到CPU资源
- 运行: 当就绪的线程被调度并获得CPU资源时,便进入运行状态，run()方法定义了线程的操作和功能
- 阻塞: 在某种特殊情况下，被人为挂起或执行输入输出操作时，让出 CPU并临时中止自己的执行，进入阻塞状态
- 死亡: 线程完成了它的全部工作或线程被提前强制性地中止或出现异常导致结束

![image-20220508174831651](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202205081748754.png)

## 线程通信

* wait() || await(): 执行此方法,当前线程就会进入阻塞状态,并释放锁
* notify() || singal(): 执行此方法,就会唤醒被wait()的一个线程.如果有多个,就唤醒优先级较高的那个
* notifyAll() || singalAll(): 执行此方法,就会唤醒所有被wait()的线程

说明:

1. wait(),notify(),notifyAll()三个方法必须使用在同步代码块或同步方法中
2. wait(),notify(),notifyAll()三个方法的调用者必须是同步代码块或同步方法中的监视器

虚假唤醒:

多个线程通信时, wait() 应该放在while循环里, 不然可能出现虚假唤醒(因为wait()在哪里睡哪里醒)

## 锁

当执行monitor enterl时，如果目标锁对象的计数器为零，那么说明它没有被其他线程所持有，Java虚拟机会将该锁对象的持有线程设置为当前线程，并且将其计数器加1。

在目标锁对象的计数器不为零的情况下，如果锁对象的持有线程是当前线程，那么Java虚拟机可以将其计数器加1，否则需要等待，直至持有线程释放该锁。

当执行monitor exit时，Java虚拟机则需将锁对象的计数器减1。计数器为零代表锁已被释放。

| ObjectMonitor.hpp关键属性 | 作用                              |
| ------------------------- | --------------------------------- |
| _owner                    | 指向持有ObjectMonitor对象的线程   |
| _WaitSet                  | 存放处于wait状态的线程队列        |
| _EntryList                | 存放处于等待锁block状态的线程队列 |
| _recursions               | 锁的重入次数                      |
| _count                    | 用来记录该线程获取锁的次数        |



- 公平锁和非公平锁

  - 非公平锁: 抢占式, 效率高, 能更充分的利用CPU的时间片，尽量减少CP0空闲状态时间。

  - 公平锁: 非抢占式, 效率低

    ```java
    private ReentrantLock lock = new ReentrantLock(true);//默认false  读写锁:new ReentrantReadWriteLock
    try {		//lock需要手动启动和结束
        lock.lock();
        ....//同步代码
    } finally {
        lock.unlock();
    }
    ```

    

- 可重入锁(递归锁)

  指在**同一个线程**在外层方法获取锁的时候，再进入该线程的内层方法会**自动获取锁**(前提，锁对象得是同一个对象)，不会因为之前已经获取过还没释放而阻塞。避免死锁

  加锁和解锁次数要一致, 否则会导致第二个线程无法获取到锁, 一直在等待

- 死锁及排查

  1. `jps -l `类似linux `ps -ef``
  2. `jstack [id]` jvm自带堆栈跟踪工具

- 读锁(独占锁) / 写锁(共享锁)

- 自旋锁SpinLock

## 线程中断LockSupport

**中断机制**

- 一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止，Thread.stop,Thread.suspend, Thread.resume都已经被废弃了。
- 在Java中没有办法立即停止一条线程，然而停止线程却尤为重要，如取消一个耗时操作因此，Java提供了一种用于停止线程的协商机制—中断，也即**中断标识协商机制**。
- 若要中断一个线程，你需要手动调用该线程的interrupt方法，该方法也仅仅是将线程对象的中断标识设成true;

**中断三大方法**

- void interrupt() : 实例方法interrupt()仅仅是设置线程的中断状态为true，发起一个协商而不会立刻停止线程
- static boolean interrupted() : 判断线程是否被中断并清除当前中断状态。
- bloolean isInterrupted() : 判断当前线程是否被中断（通过检查中断标志位)

**LockSupport**

- park(): permit许可证默认没有不能放行，所以一开始调park()方法当前线程就会阻塞，直到别的线程给当前线程的发放permit，park方法才会被唤醒。
- unpark(): 调用unpark(thread)方法后，就会将thread线程的许可证permit发放，会自动唤醒park线程，即之前阻塞中的LockSupport park()方法会立即返回.

# JMM与volatile

> JMM的关键技术点都是围绕多线程的**原子性**、**可见性**和**有序性**展开的
>
> 1. 通过JMM来实现线程和主内存之间的抽象关系。
> 2. 屏蔽各个硬件平台和操作系统的内存访问差异以实现让Java程序在各种平台下都能达到一致的内存访问效果。

- 可见性
- 原子性
- 有序性 


**happens-before**

1. 次序规则: 一个线程内，按照代码顺序，写在前面的操作先行发生于写在后面的操作;
2. 锁定规则: 一个unLock操作先行发生于后面((这里的“后面”是指时间上的先后))对同一个锁的lock操作;
3. volatile变量规则: 对一个volatile变量的写操作先行发生于后面对这个变量的读操作，**前面的写对后面的读是可见的**，这里的“后面”同样是指时间上的先后。
4. 传递规则: 如果操作A先行发生于操作B，而操作B又先行发生于操作c，则可以得出操作A先行发生于操作C;
5. 线程启动规则(Thread Start Rule): Thread对象的start()方法先行发生于此线程的每一个动作
6. 线程中断规则(Thread lnterruption Rule): 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生;
7. 线程终止规则(Thread Termination Rule): 线程中的所有操作都先行发生于对此线程的终止检 测，我们可以通过isAlive()等手段检测线程是否已经终止执行。
8. 对象终结规则(Finalizer Rule): 一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始  

## 

> volatile关键字的作用主要有如下两个：
>
> 1. 线程的可见性：当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。
> 2.  顺序一致性：禁止指令重排序

- 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新回主内存中。
- 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，重新回到主内存中读取最新共享变量

## **内存屏障--(保证可见性)**

>  内存屏障（也称内存栅栏，屏障指令等，是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作），避免代码重排序。内存屏障其实就是一种JVM指令，Java内存模型的重排规则会要求Java编译器在生成JVM指令时插入特定的内存屏障指令，通过这些内存屏障指令，volatile实现了Java内存模型中的可见性和有序性(禁重排)，但volatile无法保证原子性。
>
> - 内存屏障之前的所有写操作都要回写到主内存
> - 内存屏障之后的所有读操作都能获得内存屏障之前的所有写操作的最新结果(实现了可见性)。
> - 因此重排序时，不允许把内存屏障之后的指令重排序到内存屏障之前。
> - 对一个volatile变量的写,先行发生于任意后续对这个volatile变量的读，也叫写后读。

粗分2种:

- 读屏障(Load Barrier): 在读指令之前插入读屏障，让工作内存或CPU高速缓存当中的缓存数据失效，重新回到主内存中获取最新数据
- 写屏障(Store Barrier): 在写指令之后插入写屏障，强制把写缓冲区的数据刷回到主内存中



细分4种(cpp方法)

| 屏障类型   | 指令示例                   | 说明                                                         |
| ---------- | -------------------------- | ------------------------------------------------------------ |
| LoadLoad   | Load1; LoadLoad; Load2     | 保证load1的读取操作在load2及后续读取操作之前执行             |
| StoreStore | Store1; StoreStore; Store2 | 在store2及其后的写操作执行前，保证store1的写操作已刷新到主内存 |
| LoadStore  | Load1; LoadStore; Store2   | 在stroe2及其后的写操作执行前，保证load1的读操作已读取结束    |
| StoreLoad  | Store1; StoreLoad; Load2   | 保证store1的写操作已刷新到主内存之后，load2及其后的读操作才能执行 |

|                                                  |                                                     |
| ------------------------------------------------ | --------------------------------------------------- |
| 在每个volatile 读操作的后面插入一个LoadLoad屏障  | 禁止处理器把上面的volatile读与下面的普通读重排序。  |
| 在每个volatile 读操作的后面插入一个LoadStore屏障 | 禁止处理器把上面的volatilei读与下面的普通写重排序。 |

![image-20221126115150351](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221126115150351.png)

|                                                    |                                                              |
| -------------------------------------------------- | ------------------------------------------------------------ |
| 在每个volatile 写操作的前面插入一个 StoreStore屏障 | 可以保证在volatile写之前，其前面的所有普通写操作都已经刷新到主内存中 |
| 在每个volatile 写操作的后面插入一个 StoreLoad 屏障 | 作用是避免volatile写与后面可能有的volatile读/写操作重排序    |




![image-20221126115451821](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221126115451821.png)

## 读写过程--(没有原子性)

Java内存模型中定义的8种每个线程自己的工作内存与主物理内存之间的原子操作:

read(读取)→load(加载)→use(使用)→assign(购值)→store(存储)→write(写入)→lock(锁定)→unlock(解锁)

![image-20221126154850903](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221126154850903.png)

不具备原子性: 如果第二个线程在第一个线程读取旧值和写回新值期间读取i的域值，那么第二个线程就会与第一个线程一起看到同一个值，

在不符合以下两条规则的运算场景中，我们仍然要通过加锁来保证原子性: 

- 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。
- 变量不需要与其他的状态变量共同参与不变约束。

**结论: volatile不适合参与到依赖到当前值计算的运算 如: i++**

## 指令禁重排--(保证一致性)

> 重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段，有时候会改变程序语句的先后顺序不存在数据依赖关系，可以重排序;
>
> 存在数据依赖关系，禁止重排序-----**通过内存屏障实现**
>
> ![image-20221125153711494](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221125153711494.png)

## volatile使用场景

- 单一赋值可以，but含复合运算赋值不可以(i++之类)
- 状态标志，判断业务是否结束
- 开销较低的读，写锁策略
- DCL双端锁的发布

# CAS

> CAS的全称为Compare-And-Swap，它是一条CPU并发原语。
>
> 它包含三个操作数——内存位置、预期原值及更新值。
>
> 使用Atomicxxx(原子操作类)实现

- 执行CAS操作的时候，将内存位置的值与预期原值比较;

  - 如果相匹配，那么处理器会自动将该位置值更新为新值，
  - 如果不匹配，处理器不做任何操作，多个线程同时执行CAS操作只有一个会成功。
  - 当它重来重试的这种行为成为----**自旋**! !

  ![image-20221127093626568](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221127093626568.png)

- CAS是JDK提供的**非阻塞原子性**操作，它通过硬件保证了比较-更新的原子性。
- CAS是一条CPU的原子指令(cmpxchg指令），不会造成所谓的数据不一致问题
- 执行cmpxchg指令的时候，会判断当前系统是否为多核系统，如果是就给**总线加锁**，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作
- CAS的原子性实际上是CPU实现独占的，比起synchronized重量级锁，排他时间要短很多

## UnSafe类

- UnSage类是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问
- Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。
- Unsafe类存在于sun.misc包中，其内部方法操作可以像C的指针一样直接操作内存
- Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务
- 变量value用volatile修饰，保证了多线程之间的内存可见性。

## 自旋锁

> AtomicReference<Thread>实现

- CAS是实现自旋锁的基础，自旋翻译就是循环，一般是用一个无限循环实现。
- 这样一来，一个无限循环中，执行一个CAS操作，
  - 当操作成功返回true 时，循环结束;
  - 当返回false时，接着执行循环，继续尝试CAS操作，直到返回true。

**缺点**

- 循环时间长，开销很大
- ABA问题 ----AtomicStampedReference(V initialRef, int initialStamp)解决

## 原子操作类

- 基本类型
  - AtomicInteger
  - AtomicBoolean
  - AtomicLong

- 数组类型
  - AtomicIntegerArray
  - AtomicLongArray
  - AtomicReferenceArray

- 引用类型
  - AtomicReference
  - AtomicStampedReference
  - AtomicMarkableReference

- 对象的属性修改原子类
  - AtomicIntegerFieldUpdater
  - AtomicLongFieldUpdater
  - AtomicReferenceFieldUpdater

- 原子操作增强类---减少乐观锁重试次数

  - DoubleAccumulator
  - DoubleAdder
  - LongAccumulator
  - LongAdder----striped64实现

## Striped64

> ![image-20221127175158949](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221127175158949.png)

- LongAdder的基本思路就是分散热点，将value值分散到一个Cell数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样热点就被分散了，冲突的概率就小很多。
- sum()会将所有Cell数组中的value和base累加作为返回值，核心的思想就是将之前AtomicLong一个value的更新压力分散到多个value中去从而降级更新热点。

![image-20221127175106184](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221127175106184.png)

**具体实现**

![image-20221128095958945](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221128095958945.png)

1. 如果Cells表为空，尝试用CAS更新base字段，成功则退出;
2. 如果Cells表为空，CAS更新base字段失败，出现竞争，uncontended为true，调用longAccumulate;
3. 如果Cells表非空，但当前线程映射的槽为空，uncontended为true，调用longAccumulate;
4. 如果Cells表非空，且前线程映射的槽非空，CAS更新Cell的值，成功则返回，否则，说明槽位不够用, uncontended设为false，调用**longAccumulate**,

**longAccumulate()**

> striped64中一些变量或者方法的定义:
>
> - base: 类似于AtomicLong中全局的value值。在没有竞争情况下数据直接累加到base上，或者cells扩容时，也需要将数据写入到base上
> - collide: 表示扩容意向，false一定不会扩容，true可能会扩容。
> - cellsBusy: 初始化cells或者扩容cells需要获取锁，0:表示无锁状态 1:表示其他线程已经持有了锁
>   心
> - casCellsBusy(): 通过CAS操作修改cellsBusy的值，CAS成功代表获取锁，返回true
> - NCPU: 当前计算机CPU数量，Cell数组扩容时会使用到
> - getProbe(): 获取当前线程的hash值
> - advanceProbe(): 重置当前线程的hash值

![image-20221128131235040](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221128131235040.png)

- CASE1: Cell[ ] 数组已经初始化

  ![image-20221128133205437](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221128133205437.png)

  ​	![image-20221128133709562](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221128133709562.png)	

- CASE2: Cell[ ] 数组未初始化(首次新建)

- CASE3: Cell[ ] 数组正在初始化中

# ThreadLocal

> ThreadLocal提供线程局部变量。这些变量与正常的变量不同，因为每一个线程在访ThreadLocal实例的时候（通过其getaset方法）都有自己的、独立初始化的变量副本。
>
> ThreadLocal实例通常是类中的私有静态字段，使用它的目的是希望将状态（例如，用户ID或事务ID）与线程关联起来。

## 源码分析

![image-20221128174850836](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221128174850836.png)

当为threadLocal变量赋值，实际上就是以当前threadLocal实例为key，值为value 的Entry往threadLocalMap中存放

![image-20221128174757027](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221128174757027.png)

<span style="color:red">JVM内部维护了一个线程版的Map<ThreadLocal,Value></span>(通过ThreadLocal对象的set方法，结果把ThreadLocal对象自己当做key,放进了ThreadLoalMap中),<span style="color:red">每个线程要用到这个T的时候，用当前的线程去Map里面获取</span>，通过这样让每个线程都拥有了自己独立的变量，人手一份，竞争条件被彻底消除，在并发模式下是绝对安全的变量。

**弱引用**

![image-20221129120358011](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221129120358011.png)

ThreadLocal是一个壳子，真正的存储结构是ThreadLocal里有ThreadLocalMap这么个内部类，每个Thread对象维护着一个ThreadLocalMlap的引用，ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储。

## 内存泄漏

- ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用引用他，那么系统gc的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的ralue，如果当前线程再迟迟不结束的话(比如正好用在线程池)，这些key为null的Entry的value就会一直存在一条强引用链
- 虽然弱引用，保证了key指向的ThreadLocal对象能被及时回收，但是v指向的value对象是需要ThreadLocalMap调用get、setlt发现ke y's u时t才会去回收整个entry、value，因此弱引用不能100%保证内存不泄露。我们要在不使用某个ThreadLocal对象后，手动调用remove()方法来删除它
- set、 get方法会去检查所有键为null的Entry对象设置、获取方法会去检查所有键为空的条目对象,并调用expungestaleEntry清除value

# 对象内存布局

> 在HotSpot虚拟机里，对象在堆内存中的存储布局可以划分为三个部分:对象头(Header)、实例数据（ Instance Data）和对齐填充（Padding) 。
>
> ![image-20221129135628729](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221129135628729.png)

## 对象头

![image-20221129140750165](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221129140750165.png)

**对象标记 Mark Word**



![image](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221129142524944.png)

**类元信息 (类型指针)**

## 实例数据

## 对齐填充

保证8个字节的倍数

# Synchronized与锁升级

> **Monitor与java对象以及线程关联**
>
> 1. 如果一个java对象被某个线程锁住，则该java对象的Mark Word字段中LockWord指向monitor的起始地址
> 2. Monitor的Owner字段会存放拥有相关联对象锁的线程id
>
> ![imagine](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221129150316241.png)

![img](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/2575225-20220426174452302-1618302519.png)

- 偏向锁: MarkWord存储的是偏向的线程ID;
- 轻量锁: MarkWord存储的是指向线程栈中Lock Record的指针;
- 重量锁: MarkWord存储的是指向堆中的monitor对象的指针;

| 锁       | 优点                                                         | 缺点                                          | 适用场景                                        |
| -------- | ------------------------------------------------------------ | --------------------------------------------- | ----------------------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗、和执行非同步方法相比仅存在纳秒级的差距 | 如果线程间存在锁竞争,会带来额外的锁撤销的消耗 | 适用于只有一个线程访问同步块场景                |
| 轻量级锁 | 竞争的线程不会阻塞,提高了程序的响应速度                      | 如果始终得不到锁竞争的线程,使用自旋会消耗CPU  | 追求响应时间<br/>同步块执行速度非常快追求吞吐量 |
| 重量级锁 | 线程竞争不使用自旋,不会消耗CPU                               | 线程阻塞,响应时间缓慢                         | 同步块执行速度较长                              |

## 偏向锁: 单线程竞争

> 偏向锁会偏向于第一个访问锁的线程，如果在接下来的运行过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将永远不需要触发同步。
>
> 也即偏向锁在资源没有竞争情况下消除了同步语句，懒的连CAS操作都不做了，直接提高程序性能

- 当线程A第一次竞争到锁时，通过操作修改Mark Word中的偏向线程ID、偏向模式。如果不存在其他线程竞争，那么持有偏向锁的线程将永远不需要进行同步。
- 当一段同步代码一直被同一个线程多次访问，由于只有一个线程那么该线程在后续访问时便会自动获得锁

**使用**

偏向锁1.6后默认开启, 4秒延迟, 关闭后直接进入轻量级锁

**偏向锁的撤销**

- 偏向锁使用一种等到竞争出现才释放锁的机制，只有当其他线程竞争锁时，持有偏向锁的原来线程才会被撤销。
- 撤销需要等待全局安全点(该时间点上没有字节码正在执行)，同时检查持有偏向锁的线程是否还在执行

**升级**

- 第一个线程正在执行synchronized方法(处于同步块)，它还没有执行完，其它线程来抢夺，该偏向锁会被取消掉并出现锁升级。此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁.
- 第一个线程执行完成synchronized方法(退出同步块)，则将对象头设置成无锁状态并撤销偏向锁，重新偏向。

![image-20221130145126162](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221130145126162.png)

## 轻量级锁: 多线程竞争

> 任意时刻最多只有一个线程竞争，即不存在锁竞争太过激烈的情况，也就没有线程阻塞。

**轻量级锁的加锁**

- JVM会为每个线程在当前线程的栈帧中创建用于存储锁记录的空间，官方称为Displaced Mark Word。
- 若一个线程获得锁时发现是轻量级锁，会把锁的MarkWord复制到自己的Displaced Mark Word里面。然后线程尝试用CAS将锁的MarkWord替换为指向锁记录的指针。
  - 如果成功，当前线程获得锁，
  - 如果失败，表示Mark Word已经被替换成了其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使用自旋来获取锁。
- 自旋CAS:不断尝试去获取锁，能不升级就不往上捅，尽量不要阻塞

**轻量级锁的释放**

- 在释放锁时，当前线程会使用CAS操作将Displaced Mark Word的内容复制回锁的Mark Word里面。
  - 如果没有发生竞争，那么这个复制的操作会成功。
  - 如果有其他线程因为自旋多次导致轻量级锁升级成了重量级锁，那么CAS操作会失败，此时会释放锁并唤醒被阻寒的线程。

**锁升级**

自旋达到一定次数和程度: 

- java6之前默认自旋10次或者自旋线程数超过cpu核数一半  -XX: PreBlockSpin= 10修改
- java6之后自适应

## 重量级锁

> Java中synchronized的重量级锁，是基于进入和退出Monitor对象实现的。在编译时会将同步块的开始位置插入monitor enter指令，在结束位置插入monitor exit指令。
>
> 当线程执行到monitor enter指令时，会尝试获取对象所对应的Monitor所有权，如果获取到了，即获取到了锁，会在Monitor的owner中存放当前线程的id，这样它将处于锁定状态，除非退出同步块，否则其他线程无法获取到这个Monitor。

锁升级为轻量级或重量级锁后，Mark Word中保存的分别是线程栈帧里的锁记录指针和重量级锁指针，已经没有位置再保存哈希码,GC年龄

**当一个对象已经计算过一致性哈希码后，它就再也无法进入偏向锁状态了, 而当一个对象当前正处于偏向锁状态，又收到需要计算其一致性哈希码请求时, 它的偏向状态会被立即撤销, 并且锁会膨胀为重量级锁。**

- 在无锁状态下，Mark Word中可以存储对象的identity hash code值。当对象的hashCode()方法第一次被调用时，JVM会生成对应的identity hash code值并将该值存储到Mark Word中。
- 对于偏向锁，在线程获取偏向锁时，会用Thread lD和epoch值覆盖identity hash code所在的位置。**( 已经计算过hash code升级到轻量级, 过程中需要计算, 升级到重量级再计算)** 
- 升级为轻量级锁时，JVM会在当前线程的栈帧中创建一个锁记录(Lock Record)空间，用于存储锁对象的Mark Word拷贝，该拷贝中可以包含identity hash code，所以轻量级锁可以和identity hash code共存，哈希码和GC年龄自然保存在此，释放锁后会将这些信息写回到对象头。
- 升级为重量级锁后，Mark Word保存的重量级锁指针，代表重量级锁的ObjectMonitor类里有字段记录非加锁状态下的Mark.Nord，锁释放后也会将信息写回到对象头。



## JIT编译器对锁的优化

> Just In Time Compiler，一般翻译为即时编译器

锁消除

锁粗化

# AQS

>  AbstractQueuedSynchronizer( 抽象的队列同步器 )简称为AQS
>
> 是用来实现锁或者其它同步器组件的公共基础部分的抽象实现，是重量级基础框架及整个JUC体系的基石，主要用于解决锁分配给"谁"的问题
>
> 整体就是一个抽象的FIFO队列来完成资源获取线程的排队工作，并通过一个int类变量表示持有锁的状态

![image-20221201134323004](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221201134323004.png)

- **AQS中的队列是CLH变体的虚拟双向队列(FIFO)**	     state变量+CLH  	CLH: Craig、Landin and Hagersten队列，是个单向链表，

![image-20221201135403934](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221201135403934.png)

![image-20221201155452019](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221201155452019.png)



## 源码分析

- 公平锁:公平锁讲究先来先到，线程在获取锁时，如果这个锁的等待队列中已经有线程在等待，那么当前线程就会进入等待队列中;

- 非公平锁:不管是否有等待队列，如果可以获取锁，则立刻占有锁对象。也就是说队列的第一个排队线程苏醒后，不一定就是排头的这个线程获得锁，它还是需要参加竞争锁（存在线程竞争的情况下)，后来的线程可能不讲武德插队夺锁了。

  ![image-20221201154347404](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221201154347404.png)

整个 ReentrantLock 的加锁过程,可以分为三个阶段:

1. 尝试加锁;
2. 加锁失败,线程入队列;
3. 线程入队列后,进入阻塞状态。
4. [AQS流程图详解](https://www.bilibili.com/video/BV1ar4y1x727?p=153&amp;vd_source=64b2afb3dfa78b094350fed842fb1025)

## JUC辅助类

- CountDownLatch (减少计数) : 不减少到零不执行

- CyclicBarrier (循环栅栏): 不增加到指定不停止

- Semaphore (信号灯)  : 设置许可位, 获取许可位才能运行, 释放后可被其他线程抢占

- BlockingQueue 阻塞队列：　
  - 当队列是空的，从队列中获取元素的操作将会被阻塞
  - 当队列是满的，从队列中添加元素的操作将会被阻塞

# 读写锁

> ReentrantReadWriteLock
>
> 读写锁定义为:一个资源能够被多个读线程访问，或者被一个写线程访问，但是不能同时存在读写线程。

## 锁降级

1. 如果同一个线程持有了写锁，在没有释放写锁的情况下，它还可以继续获得读锁。这就是写锁的降级，降级成为了读锁。
2. 规则惯例，先获取写锁，然后获取读锁，再释放写锁的次序。
3. 如果释放了写锁，那么就完全转换为读锁。

锁降级遵循: 获取写锁→再获取读锁→再释放写锁的次序，写锁能够降级成为读锁。

![image-20221202170935506](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20221202170935506.png)

## 锁饥饿

> ReentrantReadWriteLock实现了读写分离，但是一旦读操作比较多的时候，想要获取写锁就变得比较困难了，因为当前有可能会一直存在读锁，而无法获得写锁

解决:　

- 使用“公平"策略可以一定程度上缓解这个问题, 但是“公平"策略是以牺牲系统吞吐量为代价的
- StampedLock类的乐观读锁
  - StampedLock采取乐观获取锁后，其他线程尝试获取写锁时不会被阻塞，在获取乐观读锁后，还需要对结果进行校验。

**StampedLock**

- 所有获取锁的方法，都返回一个邮戳(Stamp), stamp为零表示获取失败，其余都表示成功;
- 所有释放锁的方法，都需要一个邮戳〈Stamp）,这个Stamp必须是和成功获取锁时得到的Stamp一致;
- StampedLock是不可重入的，危险 (如果一个线程已经持有了写锁，再去获取写锁的话就会造成死锁)
- StampedLock有三种访问模式
  - Reading（读模式悲观）:功能和ReentrantReadWriteLock的读锁类似
  - Writing (写模式）:功能和ReentrantRedWriteLock的写锁类似
  - Optimistic readina （乐观读模式):无锁机制，类似于数据库中的乐观锁，支持读写并发，很乐观认乎读取时没人修改，假如被修改再实现升级为悲观读模式





 
