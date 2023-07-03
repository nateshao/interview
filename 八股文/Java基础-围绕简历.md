Java基础-围绕简历

# **线程池**

## 为什么要多线程

1. 发挥多核CPU 的优势，多线程可以提高CPU利⽤率和响应速度。
2. 利用好多线程机制可以提高系统的并发能力。
3. 改善程序结构。将一个复杂的的进程分为多个线程，减少类之间的耦合。

**并发编程可能会遇到很多问题，⽐如：内存泄漏、上下⽂切换、死锁 。**

## 什么是线程死锁？

1. **互斥条件**：该资源任意⼀个时刻只由⼀个线程占⽤。
2. **请求与保持条件**：⼀个进程因请求资源⽽阻塞时，对已获得的资源保持不放。
3. **不剥夺条件**：线程已获得的资源在末使⽤完之前不能被其他线程强⾏剥夺，只有⾃⼰使⽤完毕后才释放资源。 
4. **循环等待条件**：若⼲进程之间形成⼀种头尾相接的循环等待资源关系。

**为了避免死锁，只要破坏产⽣死锁的四个条件中的其中⼀个就可以。最重要的是要加锁，进行锁排序。**

## **为什么要用线程池?**

- **是为了减少每次获取资源的消耗，提⾼对资源的利⽤率。**

线程池的主要优势：

1. **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
3. **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## **说一下线程核心参数**

线程池的真正实现类是`ThreadPollExecutor`。通过点进去`ThreadPoolExecutor`源码就会发现：创建线程池参数有7个，其中有5个是必需的：

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=YjIxYzQwZmRlOWM2NDE3YmVlYWJhNjdlNmZhMWExOTBfODJ1azdSaThQVUliTms1ZGpwUlhSOHF1UWFOTERMSVBfVG9rZW46Ym94Y25HNVVRN24wTEVaRGJiSVphT3hjdUhMXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

- `corePoolSize`（必需）：核心线程大小。线程池一直运行，核心线程就不会停止。
- `maximumPoolSize`（必需）：线程池最大线程数量。**线程池能容纳同时执行的最大线程数。**
- `keepAliveTime`（必需）：非核心线程的心跳时间。如果非核心线程在keepAliveTime内没有运行任务，非核心线程会消亡。
- `TimeUnit`（必需）：指定 `keepAliveTime` 参数的时间单位。常用的有：TimeUnit.MILLISECONDS（毫秒）、TimeUnit.SECONDS（秒）、TimeUnit.MINUTES（分）。
- `workQueue`（必需）：任务队列。ArrayBlockingQueue, LinkedBlockingQueue等， 用来存放线程任务。
  - threadFactory（可选）：线程工厂。用于指定为线程池创建新线程的方式。
  - handler（可选）：拒绝策略。当达到最大线程数时需要执行的饱和策略。

## **线程池执行任务的流程?**

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=ODJiZmVkOWQwOWU3NzRhNWIxNGM5MTI3OGMwN2RkNjRfZXN4NExRZDBoYllXRTZFSlRjdmUwbDJYekxDcDdBdmhfVG9rZW46Ym94Y252NGJOVEU4VmRpQnRSVTFkZnlXOHlkXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

1. 线程池执行`execute`/`submit`方法向线程池添加任务，当任务小于核心线程数`corePoolSize`，线程池中可以创建新的线程。
2. 当任务大于核心线程数`corePoolSize`，就向阻塞队列添加任务。
3. 如果阻塞队列已满，需要通过比较参数`maximumPoolSize`，在线程池创建新的线程，当线程数量大于`maximumPoolSize`，说明当前设置线程池中线程已经处理不了了，就会执行饱和策略。

## **常用的JAVA线程池有哪几种类型?**

1. **定长线程池（****`FixedThreadPool`****）**：适用于控制线程最大并发数。只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。
2. **定时线程池（****`ScheduledThreadPool`****）**：适用于执行定时或周期性的任务。核心线程数量固定，非核心线程数量无限，执行完闲置 10ms 后回收，任务队列为延时阻塞队列。
3. **可缓存线程池（****`CachedThreadPool`****）**：适用于执行大量、耗时少的任务。无核心线程，非核心线程数量无限，执行完闲置 60s 后回收，任务队列为不存储元素的阻塞队列。
4. **单线程化线程池（****`SingleThreadExecutor`****）**：不适合并发以及可能引起 IO 阻塞性及影响 UI 线程响应的操作，如数据库操作、文件操作等。只有 1 个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列。

## **执行execute()方法和submit()方法的区别是什么呢?**

- `execute()`提交不需要返回值的任务，无法判**断任务是否被线程池执行成功与否**;
- `submit()`提交需要返回值的任务。线程池会返回一个`future`类型的对象，通过这个`future`对象可以判断任务是否执行成功，并且可以通过`future`的`get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用get (1ong timeout, TimeUnit unit) 方法则会阻塞当1前线程一段时间后立即返回，这时候有可能任务没有执行完。

## **源码中线程池是怎么复用线程的?**

源码中`ThreadPoolExecutor`中有个内置对象`Worker`，每个`worker`都是一个线程，`worker`线程数量和参数有关，每个`worker`会`while`死循环从阻塞队列中取数据，通过置换`worker`中`Runnable`对象，运行其`run`方法起到线程置换的效果，这样做的好处是避免多线程频繁线程切换，提高程序运行性能。

**合理线程数：**

`CPU密集型`：CPU核数 + 1个线程的线程池

`IO密集型`：

- CPU核数*2；
- CPU核数/1-阻塞系数。（阻塞系数：0.8～0.9）

**进程是资源分配的最小单位，线程是CPU调度的最小单位**

线程和进程的区别有:

1. 定义不一样，进程是执行中的一段程序，而一个进程中执行中的每个任务即为一个线程。
2. 一个线程只可以属于一个进程， 但一个进程能包含多个线程。
3. 线程无地址空间，它包括在进程的地址空间里。
4. 线程的开销或代价比进程的小。

## **创建线程的三种方式的对比?**

1. 采用实现`Runnable`. `Callable`接口的方式创建多线程。
2. 使用继承`Thread`类的方式创建多线程.
3. `Runnable`和 `Callable`的区别
   1. Callable重写的方法是call()，Runnable重写的方法是run()。
   2. Callable的任务执行后可返回值，而Runnable的任务是不能返回值的。
   3. Call方法可以抛出异常， run方法不可以。
   4. 运行Callable任务可以拿到一个Future对象， 表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。

**线程和进程一样分为五个阶段：新建、就绪、运行、阻塞、终止。**

## sleep() ⽅法和 wait() ⽅法区别和共同点?

两者都可以暂停线程的执⾏。 区别在于： **`Thread.sleep()`** **没有释放锁，****`Object.wait()`** **释放了锁**

- `wait()` 通常被⽤于线程间交互/通信， `sleep()`通常被⽤于暂停执⾏。 
- `wait()` 被调⽤后，线程不会⾃动苏醒，需要别的线程调⽤同⼀个对象上的 `notify()` 或者 `notifyAll()` ⽅法。 `sleep()` ⽅法执⾏完成后，线程会自动苏醒。或者可以使⽤ `wait(long timeout)`超时后线程会⾃动苏醒。

## 调⽤ start() ⽅法时会执⾏ run() ⽅法，为什么我 们不能直接调⽤ run() ⽅法？

**调⽤ start() ⽅法⽅可启动线程并使线程进⼊就绪状态，直接执⾏ run() ⽅法的话不会以多线程的⽅式执⾏。**

- 真正的多线程⼯作应该是：`new`⼀个`Thread`，线程进⼊了新建状态。调⽤ `start()` ⽅法，线程进⼊了就绪状态。 `start()` 会执⾏线程的相应准备⼯作，然后⾃动执⾏ `run()`⽅法的内容。
- 但是，直接执⾏ `run()` ⽅法，会把 `run()` ⽅法当成⼀个 `main` 线程下的普通⽅法去执⾏，并不会在某个线程中执⾏它，所以这并不是多线程⼯作。

## **线程阻塞的三种情况**

当线程因为某种原因放弃CPU使用权后，即让出了CPU时间片，暂时就会停止运行，知道线程进入可

运行状态( Runnable)，才有机会再次获得CPU时间片转入RUNNING 状态。一般来讲，阻塞的情况

可以分为如下三种:

**等待阻塞(Object.wait->等待队列)**

RUNNING状态的线程执行object .wait()方法后，JVM会将线程放入等待序列(waitting queue) ;

**同步阻塞(lock-> 锁池)**

RUNNING状态的线程在获取对象的同步锁时，若该同步锁被其他线程占用，则jVM将该线程放入锁池(lock pool)中:

**其他阻塞( sleep/join)**

RUNNING状态的线程执行Thread. sleep(1ong ms)或Thread. joinO方法，或发出I/O请求时，JVM会将该线程置为阻塞状态。当s1eep() 状态超时，joinO) 等待线程终止或超时.或者I/O处理完毕时，线程重新转入可运行状态( RUNNABLE ) ;

##  **线程死亡的三种方式**

1. 正常结束：run()或者call()方法执行完成后，线程正常结束。
2. 异常结束：线程抛出一一个未捕获的Exception 或Error, 导致线程异常结束。
3. 调用stop()：直接调用线程的stopO方法来结束该线程，但是一般不推荐使用该种方式，因为该方法通常容易导致死锁;

## **守护线程是什么?**

守护线程是运行在后台的一种特殊进程。它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。在Java中垃圾回收线程就是特殊的守护线程。

## **了解Fork/Join框架吗?**

Fork/join框架是Java7提供的一一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

Fork/Join框架需要理解两个点，「 分而治之」和「工作窃取算法」。

**「分而治之」**

以上Fork/Join框架的定义，就是分而治之思想的体现啦

**「工作窃取算法」**

把大任务拆分成小任务，放到不同队列执行，交由不同的线程分别执行时。有的线程优先把自己负责的任务执行完了，其他线程还在慢慢悠悠处理自己的任务，这时候为了充分提高效率，就需要工作盗窃算法啦~ 

工作盜窃算法就是，「 某个线程从其他队列中窃取任务进行执行的过程」。一般就是指做得快的线

程(盜窃线程)抢慢的线程的任务来做，同时为了减少锁竞争，通常使用双端队列，即快线程和慢线程

各在一端。

## **CAS了解吗?**

CAS：全称Compare and swap, 即比较并交换，它是一条CPU同步原语。是一种硬件对并发的支持，针对多处理器操作而设计的一种特殊指令，用于管理对共享数据的并发访问。**CAS是一种无锁的非阻塞算法的实现**。

- CAS包含了3个操作数：
  - 需要读写的内存值V
  - 旧的预期值A
  - 要修改的更新值B
- 当且仅当V的值等于A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作。（他的功能是判断内存某个位置的值是否为预期值，如果是则更改为新的值，这个过程是原子的。)
  -  CAS并发原语体现在Java语言中的`sum.misc.Unsafe`类中的各个方法。调用`Unsafe`类中的CAS方

法，JVM 会帮助我们实现出CAS汇编指令。这是一种完全依赖于硬件的功能，通过它实现了原子操作。再次强调，由于CAS是一种系统原语，原语属于操作系统用于范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，CAS是一条CPU的原子指令，不会造成数据不一致问题。

## **CAS有什么缺陷?**

**ABA问题：可以通过AtomicStampedReference解决ABA问题，**

- 并发环境下，假设初始条件是A，去修改数据时，发现是A就会执行修改。但是看到的虽然是A，中间可能发生了A变B，B又变回A的情况。此时A已经非彼A，数据即使成功修改，也可能有问题。

**循环时间长开销**

- 自旋CAS，如果一直循环执行，一直不成功，会给CPU带来非常大的执行开销。很多时候，CAS思想体现，是有个自旋次数的，就是为了避开这个耗时问题~
- 只能保证一个变量的原子操作。
- CAS保证的是对一个变量执行操作的原子性，如果对多个变量操作时，CAS目前无法直接保证操作的原子性的。

**可以通过这两个方式解决这个问题：**

- 使用互斥锁来保证原子性；
- 将多个变量封装成对象，通过AtomicReference来保证原子性。

## **ThreadLocal的实现原理**

- 每个线程都有一个属于自己的ThreadlocalMap。`ThreadLocal.ThreadLocalMap`
- `ThreadLocalMap`内部维护着Entry数组，每个Entry代表一个完整的对象，key是ThreadLocal本身，value是ThreadLocal的泛型值。每个线程在往Threadlocal里设置值的时候，都是往自己的ThreadLocalMap里存， 读也是以某个ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离。

ThreadLocal内存结构图:

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=NDYyNzcxYjQwN2U3ZjY2YWI0MDdjYTQ2YzExYjE4ZTFfTk8xcHlSeDduVDBiYmthRkM3M2dwUGp5NVpIYW0zZTFfVG9rZW46Ym94Y25mTnB0N2JEWGhYZUExdUNMN1RyMlBoXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

由结构图是可以看出:

- Thread对象中持有一个ThreadLocal.ThreadLocalMap的成员变量。
- ThreadLocalMap内部维护了Entry数组，每个Entry代表一个完整的对象，key是ThreadLocal本身，value 是ThreadLocal的泛型值。

# **JVM** 

## Java 内存模型

线程共享的： 堆，⽅法区

线程私有的： 虚拟机栈，本地⽅法栈，程序计数器

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=NDdiYTgxMzljOWZmYWY4YzA4NjkyNWU5NWRmNTg5OWVfYXRqcDFRcHU2RzVkNFpwTjJyZldzT1BxaVJHQTY1RTJfVG9rZW46Ym94Y25nVmpXQmlrcmxFWkhDSVRzdXQwR3ZkXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

## 什么情况下会发生栈内存溢出?

- 主要是在进行递归时，未完成不会释放资源。当线程请求的栈深度超过了虚拟机允许的最大深度时，会抛出`StackOverFlowError`异常。
- 可以通过调整参数`-xss`去调整`jvm`栈的大小

## 谈谈对`OOM`的认识?如何排查`OOM`的问题?

1. 无限创建线程就会发生栈的`OOM`；
2. 方法区 OOM，经常会遇到的是动态生成大量的类、jsp 等；
3. 直接内存 OOM，涉及到 `-XX:MaxDirectMemorySize` 参数和 `Unsafe` 对象对内存的申请。

**排查** **OOM** **的方法：**

1. 增加两个参数 `-XX:+HeapDumpOnOutOfMemoryError` 

-XX:HeapDumpPath=/tmp/heapdump.hprof，当 OOM 发生时自动 dump 堆内存信息到指定目录；

1. 同时 jstat 查看监控 JVM 的内存 和 GC 情况，先观察问题大概出在什么区域；
2. 使用 MAT 工具载入到 dump 文件，分析大对象的占用情况，比如 HashMap 做缓存未清理，时间长了就会内存溢出，可以把改为弱引用 。

## 如何判断一个对象是否存活?

有两种算法：**引用计数法**和**可达性分析算法。**

- **引⽤计数法：**给对象中添加⼀个引⽤计数器，每当有⼀个地⽅引⽤它，计数器就加1；当引⽤失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使⽤的。
- **可达性分析算法：**从“`GC Roots`” 的对象作为起点，从这些节点开始向下搜索，节点所⾛过的路径称为引⽤链，**当⼀个对象到 GC Roots 没有任何引⽤链相连的话，则证明此对象是不可⽤的**。

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=MTdlZjIwNGI3MTNiN2Y3YThiMTMxNWMzZWRiYTk0NDdfZTNkRXJ5cDg2ZGxZcWZheTlzaWpib1JUU1A4Tk5QSjdfVG9rZW46Ym94Y25keXllY0NycmZSUVNJZkhvbG10a2ZkXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

## 强引用-软引用-弱引用-虚引用

- **强引用**：把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即 使该对象以后永远都不会被用到 JVM 也不会回收。因此强引用是造成 Java 内存泄漏的主要原因之一。如String s = new String("ConstXiong")
- **软引用**：`SoftReference` 实现。用于维护一些可有可无的对象。只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常。
- **弱引用**：`WeakReference` 实现。相比软引用来说，要更加无用一些，它拥有更短的生命周期，当JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。
- **虚引用**：`PhantomReference` 实现是一种形同虛设的引用，在现实场景中用的不是很多，它主要用来跟踪对象被垃圾回收的活动。

**被引用的对象就不一定能存活**。看`Reference`类型，**弱引用在GC时会被回收，软引用在内存不足的时候，即****OOM****前会被回收**，但如果没有在Reference Chain中的对象就一定会被回收。

### GCroot对象

GcRoot是一个对象引用链的起点，引出它们指向的下一个节点，再以下个节点为起点，引出此节点指向的下一个结点。这样通过 GC Root 串成的一条线就叫引用链）直到所有的结点都遍历完毕,如果相关对象不在任意一个以 GC Root 为起点的引用链中，那么虚拟机就可以在内存不足的时候，回收 这个对象

## Java中的垃圾回收算法有哪些?

Java中有四种垃圾回收算法，分别是**标记清除法、标记整理法、复制算法、分代收集算法;**

### **标记清除法:**

该算法分为“**标记**”和“**清除**”阶段：

- 首先标记出不需要回收的对象，在标记完成后，回收掉没有被标记的对象。

**标记清除法**会带来两个明显的问题： 

1. 标记和清除的效率都不高。
2. 标记清除后会产⽣⼤量不连续的碎⽚。

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=NGY2OGQ3ODIyMzk3NDFiN2Y3N2FkMTI3YmZhNjNmMzlfa1dLZUtXNU9FSkNZeGRoZ3UwRVNWR2xmWTdlZWNCY3ZfVG9rZW46Ym94Y25sV09GSk94MHUzQUo2aXNNMktycXVoXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

### **标记整理法**：

- 第一步：利用可达性去遍历内存，把存活对象和垃圾对象进行标记；
- 第二步：让所有存活的对象向⼀端移动，然后直接清理掉端边界以外的内存。

特点：适用于存活对象多，垃圾少的情况；需要整理的过程，无空间碎片产生

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=YjI4NGRiMTA5NWNhMWU4ZDU2MDNjMWE3Y2RhZDQ1ZGNfMUhsa0NGOGFuWng3bEV6RG9WSnkzZFB6ZEFxMkk0VE5fVG9rZW46Ym94Y250ZWFBQWkwZjM5T1hIWWRnTWFOT2loXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

### 复制算法:

将内存按照容量大小分为大小相等的两块，每次只使用一块，当一块使用完了，就将还存活的对象复制到另⼀块去，然后再把使⽤的空间⼀次清理掉。

特点：不会产生空间碎片。内存使用率极低。

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2Q2M2M1NzRiODdiY2MyOGEzMzY2YzIwZjBjZWM5ODlfeVhPTHNBUUp2bk9aclA3VmxjcTNJYWJ6dTFUWk14RWVfVG9rZW46Ym94Y25MRmxWMjV1WksyQ1VOUjBCakRpMnFiXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

### 分代收集算法:

当前虚拟机的垃圾收集都采⽤**分代收集算法**。根据对象存活周期的不同将内存分为⼏块。⼀般将 java 堆分为**新⽣代**和**⽼年代**，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

- **`新⽣代`**中，每次收集都会有大量对象死去，所以可以`选择复制算法`，只需要付出少量对象的复制成本就可以完成每次垃圾收集。
- **`⽼年代`**的对象存活⼏率是⽐较⾼的，⽽且没有额外的空间对它进⾏分配担保，所以通过选择“标记-清除”或“`标记-整理`”算法进⾏垃圾收集。

## **CMS和G1**垃圾回收器 

- **CMS**：**标记清除算法**实现。以获得最短回收停顿时间为目标的收集器。操作过程：**初始标记，并发标记，重新标记，并发清除**，收集结束会产生大量空间碎片。
- **G1**：**标记整理算法**实现，操作过程主要包括以下：**初始标记，并发标记，最终标记，清理阶段**。不会产生空间碎片，可以精确地控制停顿: G1将整个堆分为大小相等的多个Region (区域)，G1跟踪每个区域的垃圾大小，在后台维护一个优先级列表，每次根据允许的收集时间，优先回收价值最大的区域，已达到在有限时间内获取尽可能高的回收效率。

G1设计初衷就是替换CMS，成为一种全功能收集器。G1在JDK9之后是默认垃圾回收器。

## 什么是双亲委派模型?

1. 一个类加载器收到了类加载请求，它首先不会自己尝试去加载这个类，而是把这个请求委派给父类加载器去完成
2. 双亲委派可以防止核心类被篡改，提升系统安全性！避免重复的类加载，加快速率!

1. **通过委派的方式，可以避免类的重复加载**。当父加载器加载过某一个类时，子加载器就不会再重新加载这个类。
2. **通过双亲委派的方式，还保证了安全性**。因为Bootstrap ClassLoader在加载的时候，只会加载JAVA_HOME中的jar包里面的类，如java.lang.Integer，那么这个类是不会被随意替换的，除非有人跑到你的机器上， 破坏你的JDK。那么，就可以避免有人自定义一个有破坏功能的java.lang.Integer被加载。这样可以有效的防止核心Java API被篡改。

## 说⼀下堆内存中对象的分配的基本策略

## 说一下JVM调优的命令?

- `jps`：显示指定系统内所有的HotSpot虛拟机进程。
- `jstat`：是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
- `jmap`：用于生成heap dump文件，如果不使用这个命令，还阔以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现0OM的时候自动生成dump文件。jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆 和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。
- `jhat`：与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看。在此要注意，一般不会直接在服务器上进行分析，因为jhat是一个耗时并且耗费硬件资源的过程，一般把服务器生成的dump文件复制到本地或其他机器上进行分析。
- `jstack`: jstack用于生成java虛拟机当前时刻的线程快照。jstack来查看 各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。如果java程序 崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。

## Controller和Restcontroller有啥区别

1. 使用Controller和Restcontroller都是走mvc设计模式的，Controller最后视图渲染的时候用的是freemarker视图，Restcontroller用的是jackson视图，都是符合视图ocp原则的。

netty和tomcat有啥区别

有区别

1. 按照旧版本来讲，tomcat使用bio，netty为nio
2. tomcat作为服务器在用，netty作为底层rpc框架在用
3. 没区别：按照新版本来讲都是实现了reactor模型，所以没区别

接口响应时间长排查，有两种方式

1. 看nginx日志
2. JMeter接口响应时间。在HTTP请求中添加监听器,启动HTTP请求后，jp@gc-Response Times Over Time监听器会自动采集接口响应时间并可视化。

# 锁

## 说⼀说⾃⼰对于 synchronized 关键字的了解 

- `synchronized` 关键字解决的是多个线程之间访问资源的同步性， `synchronized`关键字可以保证被它修饰的方法或者代码块在任意时刻只能有⼀个线程执⾏。
- synchronized 有三种使用方式：可以给**实例方法，静态方法，代码块**加锁

构造方法不能使⽤ `synchronized` 关键字修饰。 **构造⽅法本身就属于线程安全的，不存在同步的构造⽅法⼀说。**

## 单例模式`synchronized`关键字的具体使⽤

双重校验锁实现对象单例（线程安全）

```Java
public class Singleton {
    // volatile关键字，除了防⽌JVM的指令重排和保证变量的可⻅性。
    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
        // 先判断对象是否已经实例过，没有实例化过才进⼊加锁代码
        if (uniqueInstance == null) {
            // 类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

## **说一下synchronized底层实现原理?**

**synchronized关键字底层原理属于** **JVM** **层⾯。**

1. `synchronized `同步语句块的实现使用`monitorenter`和 `monitorexit`指令。
   1. `monitorenter `指向同步代码块的开始位置；
   2. `monitorexit `指向同步代码块的结束位置。 
2. `synchronized`修饰的方法并没有 `monitorenter `和 `monitorexit `指令，取得代之的确实是 `ACC_SYNCHRONIZED`标识，该标识指明了该⽅法是⼀个同步⽅法。 不过两者的本质都是对对象监视器 monitor 的获取。

https://blog.csdn.net/weixin_42129005/article/details/113369940

### synchronized 同步语句块的情况

```Java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

在SynchronizedDemo目录下执行：`javac -encoding UTF-8 SynchronizedDemo.java`

通过 JDK ⾃带的 javap 命令查看 SynchronizedDemo 类的相关字节码信息：⾸先切换到类的对 应⽬录执⾏ javac SynchronizedDemo.java 命令⽣成编译后的 .class ⽂件，然后执⾏ `javap -c -s -v -l SynchronizedDemo.class`

## **synchronized锁升级的原理?**

在synchronized锁升级过程中涉及到以下几种锁。先说一下这几种锁是什么意思.

- 偏向锁：只有一个线程争抢锁资源的时候，将线程拥有者标识为当前线程.
- 轻量级锁(自旋锁)：一个或多个线程通过CAS去争抢锁，如果抢不到则一直自旋.
- 重量级锁：多个线程争抢锁，向内核申请锁资源，将未争抢成功的锁放到队列中直接阻塞.

1. **synchronized锁升级原理**：在锁对象头里面有一个`threadid`字段，在第一次访问`threadid`为空，jvm让它持有偏向锁，并将`threadid`设置为其线程id，再次进入的时候会先判断

`threadid`是否与其`线程id`一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为**轻量级锁**，通过自旋循环一定次数来获取锁，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了`synchronized`锁的升级。

1. **锁的升级的目的**：是为了减低了锁带来的性能消耗。在Java 6之后优化`synchronized`的实现方式，使用了偏向锁升级为轻量级锁再升级到重量级锁的方式，从而减低了锁带来的性能消耗。

[synchronized锁升级过程_程序员bling的博客-CSDN博客_synchronized锁升级过程](https://blog.csdn.net/wangliangluang/article/details/123463618)

## **synchronized锁能降级吗?**

可以的。

具体的触发时机：在全局安全点(`safepoint`) 中，执行清理任务的时候会触发尝试降级锁。

当锁降级时，主要进行了以下操作：

1. 恢复锁对象的`markword`对象头；
2. 重置`ObjectMonitor`, 然后将该`ObjectMonitor`放入全局空闲列表，等待后续使用。

## **volatile的使用及其原理**

volatile的两层语义:

1. **volatile保证变量对所有线程的可见性**：当volatile变量被修改，新值对所有线程会立即更新。或者理解为多线程环境下使用volatile修饰的变量的值一定是最新的。
2. **jdk1.5以后volatile完全避免了指令重排优化**，实现了有序性。（**指令重排序**是指编译器或CPU为了优化程序的执行性能而对指令进行重新排序的一种手段，重排序会带来可见性问题，所以在多线程开发中必须要关注并规避重排序。）

**volatile的原理**：

获取JIT (即时Java编译器，把字节码解释为机器语言发送给处理器)的汇编代码，发现volatile多加了

lockaddI指令，这个操作相当于一个内存屏障，使得lock指令后的指令不能重排序到内存屏障前的位

置。这也是为什么JDK1.5以后可以使用双锁检测实现单例模式。

lock前缀的另一层意义是使得本线程工作内存中的volatile变量值立即写入到主内存中，并且使得其他线程共享的该volatile变量无效化，这样其他线程必须重新从主内存中读取变量值。

https://www.javazhiyin.com/61019.html

## **synchronized 关键字和 volatile 关键字的区别？**

`synchronized`关键字和 `volatile`关键字是两个互补的存在，⽽不是对⽴的存在！ 

1. `volatile` 关键字是线程同步的轻量级实现，所以 `volatile` 性能肯定⽐ `synchronized`关键字要好。但是 **`volatile`** **关键字只能⽤于变量⽽** **`synchronized`****关键字可以修饰⽅法以及代码块。** 
2. `volatile` 关键字能保证数据的可⻅性，但不能保证数据的原⼦性。 `synchronized`关键字两者都能保证。 
3. `volatile` 关键字主要⽤于解决变量在多个线程之间的可⻅性，⽽ `synchronized`关键字解决 的是多个线程之间访问资源的同步性。

## **synchronized和Lock有什么区别?**

1. synchronized可以给类.方法.代码块加锁；而lock只能给代码块加锁。
2. synchronized不需要手动获取锁和释放锁，使用简单，发生异常会自动释放锁，不会造成死锁；而lock需要自己加锁和释放锁，如果使用不当没有unLock()去释放锁就会造成死锁。.
3. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

## **synchronized和ReentrantLock [ri:'entrənt]  区别是什么?**

两者都是可重入锁

1. 可重入锁：重入锁，也叫做递归锁，可重入锁指的是在一个线程中可以多次获取同一把锁，比如:一个线程在执行一个带锁的方法，该方法中又调用了另一个需要相同锁的方法，则该线程可以直接执行调用的方法，而无需重新获得锁，两者都是同一个线程每进入一次，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。
2. synchronized依赖于JVM；而ReentrantLock依赖于API
3. synchronized是依赖于JVM实现的，前面我们也讲到了虛拟机团队在JDK1.6为synchronized关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的
4. ReentrantLock是JDK层面实现的(也就是API层面，需要lock()和unlock()方法配合try/finally语句块来完成)
5. ReentrantLock比synchronized增加了一些高级功能相比synchronized，ReentrantLock增加了 一些高级功能。主要来说主要有三点:
   1. ①等待可中断:
   2. ②可实现公平锁;
   3. ③可实现选择性通知( 锁可以绑定多个条件)

- 等待可中断。通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- ReentrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。ReentrantLock默认情况 是非公平的，可以通过ReentrantLock类的ReentrantLock(boolean fair)构造方法来制定是否是公平的。
- ReentrantLock类线 程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。在使用notify)/notifyAll()方法进行通知时，被通知的线程是由JVM选择的，用ReentrantLock类结合Condition实例可以实现"选择性通知”

## **了解ReentrantLock吗?**

1. ReetrantLock是一个可重入的独占锁，主要有两个特性，一个是支持公平锁和非公平锁，一个是可重入。
2. ReetrantLock实现依赖于AQS(AbstractQueuedSynchronizer)。
3. ReetrantL ock主要依靠AQS维护一个阻塞队列，多个线程对加锁时，失败则会进入阻塞队列。等待唤醒，重新尝试加锁。

## **ThreadLocal是什么? ThreadLocal 了解么？**

一般我们创建的变量是可以被任何⼀个线程访问并修改的。如果想实现每⼀个线程都有自己的专属本地变量的话，可以通过`ThreadLocal`，即线程本地变量。

ThreadLocal的应用场景有

- 数据库连接池
- 会话管理中使用

## ThreadLocal 原理讲⼀下

从 Thread 类源代码⼊⼿。

```Java
public class Thread implements Runnable {  ...... //与此线程有关的ThreadLocal值。由ThreadLocal类维护 ThreadLocal.ThreadLocalMap threadLocals = null; //与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护 ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;  ...... }
```

从上⾯ Thread 类 源代码可以看出 Thread 类中有⼀个 threadLocals 和 ⼀个 inheritableThreadLocals 变量，它们都是 ThreadLocalMap 类型的变量,我们可以把 ThreadLocalMap 理解为 ThreadLocal 类实现的定制化的 HashMap 。默认情况下这两个变量都 是 null，只有当前线程调⽤ ThreadLocal 类的 set 或 get ⽅法时才创建它们，实际上调⽤这两 个⽅法的时候，我们调⽤的是 ThreadLocalMap 类对应的 get() 、 set() ⽅法。

ThreadLocal 类的 set() ⽅法

```Java
public void set(T value) {  
    Thread t = Thread.currentThread(); 
         ThreadLocalMap map = getMap(t);  
         if (map != null)  map.set(this, value);  
         else  createMap(t, value);  
     }
 ThreadLocalMap getMap(Thread t) {  
     return t.threadLocals;  
 }
```

通过上⾯这些内容，我们⾜以通过猜测得出结论：最终的变量是放在了当前线程的 ThreadLocalMap 中，并不是存在 ThreadLocal 上， ThreadLocal 可以理解为只 是 ThreadLocalMap 的封装，传递了变量值。 ThrealLocal 类中可以通过 Thread.currentThread() 获取到当前线程对象后，直接通过 getMap(Thread t) 可以访问到该线程的 ThreadLocalMap 对象。 ThreadLocal 内部维护的是⼀个类似 Map 的 ThreadLocalMap 数据结构， key 为当前对象 的 Thread 对象，值为 Object 对象。

```Java
ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {  ...... }
```

⽐如我们在同⼀个线程中声明了两个 ThreadLocal 对象的话，会使⽤ Thread 内部都是使⽤仅有 那个 ThreadLocalMap 存放数据的， ThreadLocalMap 的 key 就是 ThreadLocal 对象，value 就是 ThreadLocal 对象调⽤ set ⽅法设置的值。

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=YmI4OWNmNzgyMWVmMjkwODc2NmQ0YjNiOGEyYzMzZTZfcU9XbkxXcDdsQmhJWUdrU1hkRHlEbXhIQThsZ1ZOWjRfVG9rZW46Ym94Y25JeDBsMjJ4OGZuRjhKYmh6cWtXTTdmXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=NjJlZjFmNThlZDFiYmY3MjFmZTM1MTMwYjg3MGJjNmNfQ0VQNEFzY21jNmdTVkVra2RLRUJ4SGlPNEV4TGRWaWdfVG9rZW46Ym94Y25POHhyd0wxTjJtQ1Bqc29mVXdDVHBnXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

## ThreadLocal 内存泄露问题了解不？

`ThreadLocalMap`中使⽤的 **key 为 ThreadLocal 的弱引⽤,⽽ value 是强引⽤**。所以，如果 `ThreadLocal`没有被外部强引⽤的情况下，在垃圾回收的时候，**key 会被清理掉，⽽ value 不会被清理掉。**

这样⼀来，ThreadLocalMap中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远⽆法被 GC 回收，这个时候就可能会产⽣内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，在调⽤ set() 、 get() 、 remove() ⽅法的时候，会清理掉 key 为 null 的记录。使⽤完 `ThreadLocal`⽅法后 最好⼿动调⽤ `remove()` ⽅法

# 集合

## **1.常见的集合有哪些?**

Java集合类主要由两个根接口`Collection`和`Map`派生出来的，`Collection`派生出 了三个子接口: `List`、`Set`、`Queue `( Java5新增的队列)，因此Java集合大致也可分成`List、Set、 Queue、Map`四种接口体系。

注意: `Collection`是 一个接口，`Collections`是 一个工具类，Map不是Collection的子接口。

Java集合框架图如下:

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=MTBjYjIwZGRiOGI4NzY0ZjY2MjY5MzcyY2VmYzYwMzdfbFl4VWRBdldsaGJtZURBQkdLY2hmRFFHUzdFTkJSNFFfVG9rZW46Ym94Y25RRGF3aTljQktEMjF5MXBDMHlZWGxoXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

图中，**List代表了有序可重复集合，可直接根据元素的索引来访问: Set代表无序不可重复集合，只能根据元素本身来访问: Queue是队列集合。**

`Map`代表的是存储`key-value`对的集合，可根据元素的`key`来访问`value`。

上图中淡绿色背景覆盖的是集合体系中常用的实现类，分别是`ArrayList、LinkedList、ArrayQueue、HashSet、TreeSet、 HashMap、 TreeMap`等实现类。

## **2.Java中List 和 Set 的区别**

两个接口都是继承自`Collection`，是常用来存放数据项的集合，主要区别如下：

① `List`中允许插入重复的元素，而在`Set`中不允许重复元素存在。

② `List`是有序集合，会保留元素插入时的顺序，`Set`是无序集合。

③ `List`可以通过下标来访问，而`Set`不能。

## **3.线程安全的集合有哪些?线程不安全的呢?**

**线程安全的**

1. `Hashtable`：比HashMap多 了个线程安全。
2. `ConcurrentHashMap`：是一 种高效但是线程安全的集合。
3. `Vector`：比Arraylist多 了个同步化机制。
4. `Stack`：栈，也是线程安全的，继承于Vector.

**线程不安全**

HashMap、Arraylist、LinkedList、HashSet、TreeSet、TreeMap

## **4. Arraylist与LinkedList异同点?**

1. **是否保证线程安全**：ArrayList和LinkedList都是不同步的，也就是不保证线程安全:
2. **底层数据结构**： Arraylist底层使用的是Object数组; LinkedList底层使用的是双向循环链表数据结构;
3. **插入和删除是否受元素位置的影响**：
   1. ArrayList采用数组存储，插入和删除元素的时间复杂度受元素位置的影响。ArrayList 在头或者末尾，时间复杂度就是0(1)。在指定位置i插入和删除元素的话( add(int index, E element)) 时间复杂度就为0(n-i)。因为在进行上述操作的时候集合中第i和第i个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
   2. LinkedList采用链表存储，插入/删除时间复杂度O (1)而数组为近似O (n)
4. **是否支持快速随机访问**: LinkedList不支持高效的随机元素访问，而ArrayList实现了RandmoAccess接口，所以有随机访间功能。快速随机访间就是通过元素的序号快速获取元素对象(对应于get(int index) 方法)。
5. **内存空间占用：**ArrayList的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间(因为要存放直接后继和直接前驱以及数据)。

## **5. ArrayList与Vector区别?**

1. **Vector是线程安全的，ArrayList不是线程安全的。**其中，Vector底层加了`synchronized`关键字，来保证线程的安全性。如果有多个线程会访问到集合，那最好是使用Vector，因为不需要我们自己再去考虑和编写线程安全的代码。
2. ArrayList在底层数组不够用时在原来的基础上扩展0.5倍， Vector是扩展1倍， 这样ArrayList就有利于节约内存空间。

## **7. HashMap的底层数据结构是什么?**

在JDK1.7和DK1.8中有所差别：

1. 在JDK1.7中，由”数组+链表”组成，数组是HashMap的主体，链表主要为了解决哈希冲突
2. 在JDK1.8中，由”数组+链表+红黑树”组成。当链表过长，则会严重影响HashMap的性能，红黑树搜索时间复杂度是O(logn)，而链表是糟糕的O(n)。因此，JDK1.8对数据结构做了进一步的优化，引入了红黑树，链表和红黑树在达到一定条件会进行转换:
   1. 当链表超过8且数据总量超过64才会转红黑树。
   2. 将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树，以减少搜索时间。

##  HashMap的扩容方式

## **8. 解决hash冲突的办法有哪些? HashMap用的哪种?**

解决Hash冲突方法有：**链地址法(拉链法)、建立公共溢出区、开放定址法、再哈希法。HashMap中采**

**用的是链地址法。**

- **链地址法(拉链法)**，将哈希值相同的元素构成一个同义词的单链表，并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。链表法适用于经常进行插入和删除的情况。建立公共溢出区，将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

## **9. 为什么在解决hash冲突的时候，不直接用红黑树?而选择先用链表，再转红黑树?**

因为红黑树需要进行左旋，右旋，变色这些操作来保持平衡，而单链表不需要。

- 当元素小于8个的时候，此时做查询操作，链表结构已经能保证查询性能。
- 当元素大于8个的时候，红黑树搜索时间复杂度是O(logn)，而链表是O(n)，此时需要红黑树来加快查询速度，但是新增节点的效率变慢了。

因此，如果一开始就用红黑树结构，元素太少，新增效率又比较慢，无疑这是浪费性能的。

## **10. HashMap的put方法流程?**

以DK1.8为例，简要流程如下:

1. 首先根据key的值计算hash值，找到该元素在数组中存储的下标;
2. 如果数组是空的，进行resize初始化;
3. 如果没有哈希冲突直接放在对应的数组下标里;
4. 如果冲突了，且key已经存在，就覆盖掉value;
5. 如果冲突后，发现该节点是红黑树，就将这个节点挂在树.上;
6. 如果冲突后是链表，判断该链表是否大于8，如果大于8并且数组容量小于64，就进行扩容；如果链表节点大于8并且数组的容量大于64，则将这个结构转换为红黑树:否则，链表插入键值对，若key存在，就覆盖掉value.

在push的时候，根据key的值计算hash值，找到该元素在数组中存储的下标；

然后去和它的数组长度减一去按位语去计算出它的一个数组下标，然后去判断当前我的这张单链表里面，它是否存在相同的K，如果存在的话就进行覆盖，如果不存在的话，就进行一个头插法，插进我的单链表里面，然后这就是完成了一个put的操作，然后在每一次put之后，会检查两个东西，一个是它的负载因子默认的，如果是，我在创建这个map对象的时候，不给定它的负载因子，就是默认的就是0.75，然后去看它的那个负载因子，有没有说超过我的默认的负载。

### hashmap的get流程

 get的时候，通过计算它的哈希值去获取到它的一个下标，然后就便利我的单元表或者是红黑数，如果说找到了，那就会返回它的value值，如果说没有找到的话，就返回一个null。

## **11. 一般用什么作为HashMap的key?**

一般用Integer，String 这种不可变类当HashMap当key，而且String最为常用。

- 因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。这就是
- HashMap中的键往往都使用字符串的原因。
- 因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非
- 常重要的,这些类已经很规范的重写了hashCode()以及equals()方法。

## **12. HashMap为什么线程不安全?**

1. **多线程下扩容死循环**。JDK1.7中的HashMap使用**头插法**插入元素，在多线程的环境下，扩容的时

候有可能导致环形链表的出现，形成死循环。因此，JDK1.8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。

1. **多线程的put可能导致元素的丢失**。多线程同时执行put操作，如果计算出来的索引位置是相同

的，那会造成前一个key被后一个key覆盖，从而导致元素的丢失。此问题在JDK 1.7和JDK 1.8中都存在。

1. **put和get并发时**，可能导致get为null。线程1执行put时，因为元素个数超出threshold而导致rehash,线程2此时执行get,有可能导致这个问题。此问题在JDK 1.7和JDK 1.8中都存在。

## **HashMap线程不安全?什么是线程安全呢？**

1. Hashtable和ConcurrentHashMap是线程安全的。
2. Hashtable并发性不如 ConcurrentHashMap
   1. Hashtable是表锁，**Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占；**
   2. ConcurrentHashMap 是分段锁，一次锁住一个桶。ConcurrentHashMap默认将hash表分为16个桶，诸如get、put、remove等常用操作只锁住当前需要用到的桶。一个线程进入，同时有16个写线程执行，并发性能大大提高。

## **ConcurrentHashMap的实现原理是什么?**

ConcurrentHashMap在JDK1.7和JDK1.8的实现方式是不同的。

1. JDK1.7采用**Segment 和HashEntry**的分段锁机制实现线程安全，其中segment继承自ReentrantLock。
2. JDK1.8 采用CAS+Synchronized实现更加低粒度的锁。保证线程安全。

将锁的级别控制在了更细粒度的哈希桶元素级别，也就是说只需要锁住这个链表头结点(红黑树的根节点)，就不会影响其他的哈希桶元素的读写，大大提高了并发度。

## **HashSet和HashMap区别?**

1. `HashSet`实现了`Set`接口, 仅存储对象; `HashMap`实现了 `Map`接口, 存储的是键值对；
2. `HashSet`底层其实是用`HashMap`实现存储的, HashSet封装了一系列HashMap的方法. 依靠HashMap来存储元素值,(利用hashMap的key键进行存储), 而value值默认为Object对象. 所以HashSet也不允许出现重复值, 判断标准和HashMap判断标准相同, 两个元素hashCode相等并且通过equals()方法返回true

# **AQS**

## **1.说一说什么是AQS?**

AQS 的全称为（ AbstractQueuedSynchronizer ），这个类在`java.util.concurrent.locks`包下⾯。

ReentrantLock、ReentrantReadWriteLock底层都是基于AQS来实现的

AQS的全称是AbstractQueuedSynchronizer，是抽象队列同步器，其实他就是一个用来构建锁和同步器的框架，内部实现的关键是：先进先出的队列、state状态，在LOCK包中的相关锁(常用的有ReentrantLock、 ReadWriteLock)都是基于AQS来构建。

## CountDownLatch 是什么

CountDownLatch字面意思是倒计时门闩，是基于AQS共享锁实现的，主要有两个方法，countDown方法和await方法。创建CountDownLatch时会指定count数量，count不为0时调用await方法线程会阻塞，每调用一次countDown方法会将count减1，直到count==0时await方法才会解除阻塞。

这个工具经常用来用来协调多个线程之间的同步，或者说起到线程之间的通信（而不是用作互斥的作用）

## **5.AQS组件了解吗?**

- Semaphore(信号量)-允许多个线程同时访问: synchronized 和ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。
- CountDownLatch ( 倒计时器) : CountDownLatch是一 个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。
- CyclicBarrier(循 环栅栏): CyclicBarrier 和CountDownLatch非常类似，它也可以实现线程间的技术等待，但是它的功能比CountDownLatch更加复杂和强大。主要应用场景和
- CountDownLatch类似。CyclicBarrier 的字面意思是可循环使用(Cyclic) 的屏障(Barrier) 。它要做的事情是，让一组线程到达一个屏障(也可以叫同步点)时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是
- CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

# **Java 基础** 

## Java和C++有什么区别?

相同点：都是面向对象的语言，都支持封装、继承和多态; 

1. C++支持指针和多继承，而Java没有指针，不支持多重继承，允许一个类实现多个接口。
2. Java自动进行无用的内存回收操作，而C++中必须由程序手动释放内存资源，增加程序员的负担。

## **Java和Go的区别是什么？**

1. Java有重载。Go没有重载，必须具有方法和函数的唯一名称；
2. Java有多态，Go没有。
3. Java不支持多继承，Go支持多继承。

## public、private、 protected、 default 的区别?

Java 支持4种不同的访问权限。

- `default`：“默认访问模式“。该模式下，只允许在同一个包中进行访问。
- `public`：对所有类可见
- `private`：在同一类内可见。**注意：不能修饰类（外部类）**
- `protected`：对同一包内的类和所有子类可见。**注意：不能修饰类（外部类）**。

## **Java中的static关键字**

`static`关键字有两个主要的作用

- static 可以用来修饰：变量、方法、内部类、代码块；
- static 修饰的属性，使用**类名.属性**访问；

## Java的8种数据类型

Java的基本数据类型有8种，分别是：

byte（位）

short（短整数）

int（整数）4个字节,32位，int的取值范围为`-2^31 ~ 2^31-1`

long（长整数）

float（单精度）

double（双精度）

char（字符）

boolean（布尔值）

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=N2E2YjY4ZTE0NzcwZGEzZThkODM2OTE2ZjM2YTVlMmNfQ2l0ODZ0S29qc01LZFFYWExrcllydnV4bVlxVzJQOE5fVG9rZW46Ym94Y25UeEhINVp3QmRwTEFjYkE0anEzb0VmXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

## JVM、JRE和JDK的关系是什么?

**JDK包含JRE，JRE包 含****JVM****。**

1. JDK是Java开发具包，bin目录下有很多.exe文件。编译器（javac）和⼯具（如 javadoc 和 jdb）
2. JRE是Java运行环境，它是运行己编译Java程序所需的所有内容的集合，包括Java虚拟机(JVM)，Java类库，java命令和其他的一些基础构件。
3. JVM是Java虚拟机，是用来执行Java字节码(二进制的形式)的虚拟机计算机；**JVM是运行在操作系统之上的，与硬件没有任何关系。**

## 字符型常量和字符串常量的区别?

|          | 字符型常量        | 字符串常量   |
| -------- | ----------------- | ------------ |
| 形式上   | 单引号            | 双引号       |
| 含义上   | 整型值( ASCII 值) | 地址值       |
| 内存⼤⼩ | 占2 个字节        | 占若⼲个字节 |

## 自动装箱与拆箱 

**装箱**：将基本类型⽤它们对应的引⽤类型包装起来。**比如int 变为 Interger**。

**拆箱**：将包装类型转换为基本数据类型；比如Interger变为int 。

## 对象相等和引用相等，两者有什么不同? 

- 对象的相等，比较的是内存中存放的**内容**是否相等。equal
- 引⽤相等，比较的是他们指向的**内存地址**是否相等。==

## **1.** ⾯向对象和⾯向过程的区别

- **⾯向对象** ：⾯向对象易维护、易复⽤、易扩展。 因为面向对象有**继承、封装、多态性**的特性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，⾯向对象性能⽐⾯向过程低。
- **⾯向过程** ：⾯向过程性能⽐⾯向对象⾼。 因为类调⽤时需要实例化，开销比较⼤，比较消耗资源，所以当性能是最重要的考量因素的时候，⽐如**单⽚机、嵌⼊式开发、Linux/Unix 等 ⼀般采⽤⾯向过程开发**。但是，**⾯向过程没有⾯向对象易维护、易复⽤、易扩展。** 

## **2. 重载(Overload)和重写(Override) 的区别是什么?**

方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。

- **重写：**⽅法名、参数列表、返回值类型必须相同
- **重载：方法名字相同，而参数不同，个数不同、顺序不同**。⽅法返回值和访问修饰符可以不同。

![img](https://bytedancecampus1.feishu.cn/space/api/box/stream/download/asynccode/?code=YmUzZmJmYzkwM2NjY2FmMGExNTcwY2ViMTk5NzE5NTRfdkZGVjYzNG1WREVSOURUOEE1R2FOQ3JQeDhoVHQ5dmxfVG9rZW46Ym94Y25qbTl3dHBqZ21jbFVDaG92b1M5OWFlXzE2ODgzNTg0NjI6MTY4ODM2MjA2Ml9WNA)

 构造器不能被 override（重写），但是可以 overload（重载）

## **3. String，StringBuffer 和 StringBuilder 的区别是什么? String 为什么是不可变的?**

**可变与可不变**

1. 点进去`String`源码里面看，里面有"`final`”修饰符， 所以`string`对象是不可变的。 private final char value[];
2. `StringBuilder`与 `StringBuffer`都继承⾃ `AbstractStringBuilder`类，在 `AbstractStringBuilder`中也是使⽤字符数组保存字符串 `char[] value` 但是没有⽤ `final`关键字修饰，所以这两种对象都是可变的。

**是否线程安全。**

1. `StringBuilder`是非线程安全的。
2. `String`中的对象是不可变的，也就可以理解为常量，显然线程安全。
3. `StringBuffer`对方法加了同步锁`synchronized`或者对调用的方法加了同步锁，所以是线程安全的。

```Java
 @override
 public synchronized StringBuffer append(String str) {
     toStringCache = nu11 ;
     super.append(str);
     return this;
 }
```

## **4. final、finally、 finalize的区别?**

1. `final`用于修饰**变量、方法和类**。
   1. `final`变量：**被修饰的变量不可变**，不可变分为引用不可变和对象不可变，`final`指的是引用不可变，`final`修饰的变量必须初始化，通常称被修饰的变量为常量。
   2. `final`方法：**被修饰的方法不允许任何子类重写**，子类可以使用该方法。
   3. `final`类：**被修饰的类不能被继承**，所有方法不能被重写。
2. `finally`**用于异常处理**的一部分，它只能在`try/catch `语句中，并且附带一个语句块表示这段语句最终一定被执行(无论是否抛出异常)，经常被用在需要释放资源的情况下，`system.exit (0)` 可以阻断`finally`执行。
3. `finalize`是在`java.lang.object`里定义的方法，也就是说每一个对象都有这么个方法，这个方法在`gc`启动，该对象被回收的时候被调用。

## **5. Java语言是如何实现多态的?**

Java实现多态有3个必要条件：**继承、重写和向上转型**。

- **继承**：在多态中必须存在有继承关系的子类和父类。
- **重写**：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
- **向上转型**：在多态中需要将子类的引用赋给父类对象，只有这样该引用才既能可以调用父类的方法，又能调用子类的方法。

```Java
 List list = new ArrayList()
```

是多态的形式，父类对象指向子类引用，这样可以有效并且灵活地随时修改ArrayList()部分，对以后程序的扩展和修改非常有好处 

```Java
public class Test {
     public static void main(String[] args) {
       show(new Cat());  // 以 Cat 对象调用 show 方法
       show(new Dog());  // 以 Dog 对象调用 show 方法
                 
       Animal a = new Cat();  // 向上转型  
       a.eat();               // 调用的是 Cat 的 eat
       Cat c = (Cat)a;        // 向下转型  
       c.work();        // 调用的是 Cat 的 work
   }  
             
     public static void show(Animal a)  {
       a.eat();  
         // 类型判断
         if (a instanceof Cat)  {  // 猫做的事情 
             Cat c = (Cat)a;  
             c.work();  
         } else if (a instanceof Dog) { // 狗做的事情 
             Dog c = (Dog)a;  
             c.work();  
         }  
     }  
 }
  
 abstract class Animal {  
     abstract void eat();  
 }  
   
 class Cat extends Animal {  
     public void eat() {  
         System.out.println("吃鱼");  
     }  
     public void work() {  
         System.out.println("抓老鼠");  
     }  
 }  
   
 class Dog extends Animal {  
     public void eat() {  
         System.out.println("吃骨头");  
     }  
     public void work() {  
         System.out.println("看家");  
     }  
 }
```

## **6. 接口和抽象类的区别**

① 抽象类和接口都不能被实例化。

② 抽象类和接口都可以定义抽象方法，子类/实现类必须覆写这些抽象方法。

|              | 接口interface                        | 抽象类abstract                   |
| ------------ | ------------------------------------ | -------------------------------- |
| 区别1        | 接口是多继承                         | 抽象类是单继承                   |
| 区别2        | 接口中属性的访问控制符只能是 public  | 抽象类中的属性访问控制符无限制   |
| 设计层面来说 | 接口是行为的抽象，是一种行为的规范。 | 抽象是对类的抽象，是一种模板设计 |

## **7. 抽象类能使用final修饰吗?**

不能，定义抽象类就是让其他类继承的，如果定义为final该类就不能被继承，这样彼此就会产生矛盾，所以final不能修饰抽象类

## **8. java创建对象有哪几种方式?**

java中提供了以下四种创建对象的方式:

1. **new新对象**
2. **反射机制**
3. **clone机制**
4. **序列化机制**

对于clone机制，需要注意**浅拷贝和深拷贝**的区别；在java中序列化可以通过实现`Externalizable`或者`Serializable`来实现。

## **反射有什么好处**

**一、反射的好处**：可以创建任意类的对象，可以执行任意方法 前提：不改变该类的任何代码。可以创建任意类的对象，可以执行任意方法 在不使用注解的前提下只需要修改配置文件不需要修改main函数中的代码就可以实现调用其他类的前提

https://blog.csdn.net/c3476741954/article/details/105150738

## **9. 浅拷贝和深拷贝的区别**

**浅拷贝**：**浅拷贝只拷贝某个对象的引用，而不拷贝对象本身，新旧对象还是共享同一块内存**。如果原地址内容发生改变，浅拷贝出的对象也会相应改变。**Object.clone方法是浅拷贝**

**深拷贝**：深拷贝的**新对象和原对象不共享内存**，修改新对象不会改变原对对象。深拷贝相比于浅拷贝速度较慢并且花销较大。**Java的序列化是深拷贝**。

## **10. 序列化与反序列化**

1. Java 序列化是指把 Java 对象转换为字节序列的过程；
2. Java 反序列化是指把字节序列恢复为 Java 对象的过程；
3. 不想进行序列化的变量，使⽤ `transient`关键字修饰。

**Java 序列化和反序列化好处**：

1. 实现了数据的持久化，通过序列化可以把数据永久的保存在硬盘上；
2. 利用序列化实现远程通信，即在网络上传递对象的字节序列。（序列化与反序列化则实现了 **进程通信间的对象传送**，发送方需要把这个Java对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为Java对象）

1. ## **值传递和引用传递?为什么说Java中只有值传递?**

**值传递**：方法调用时，传递的是值的拷贝，也就是说传递后就互不相关了。

**引用传递**：方法调用时，传递的是引用的地址。传递前和传递后都指向同一个引用(同一个内存空间)。

**基本类型作为参数被传递时肯定是值传递**；引用类型作为参数被传递时也是值传递，只不过"值"为对应的引用。

```Java
 public class ReferencePkValue2 {
      
     public static void main(String[] args) { 
         ReferencePkValue2 t = new ReferencePkValue2(); 
         int a = 99; 
         t.test1(a);//这里传递的参数a就是按值传递 
         System.out.println(a);
          
         MyObj obj = new MyObj(); 
         t.test2(obj);//这里传递的参数obj就是引用传递
         System.out.println(obj.b);
     } 
      
     public void test1(int a){ 
         a = a++;
         System.out.println(a);
     } 
      
     public void test2(MyObj obj){ 
         obj.b = 100;
         System.out.println(obj.b);
     }
 }
```

Java中方法参数传递方式是**按值传递**。

如果参数是**基本类型**，传递的是基本类型的字面量值的拷贝。

如果参数是**引用类型**，传递的是该参量所引用的对象在堆中地址值的拷贝。

## Object 有哪些常用方法？

Object 是所有类的根，是所有类的父类，所有对象包括数组都实现了 Object 的方法。

```Java
String s1 = new String("123");
String s2 = new String("123");
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));

Object obj = new Object();
obj.equals(s1); // 比较两个对象内容是否相等
obj.hashCode(); // 获取哈希码
obj.notify();   // 多线程中唤醒功能
obj.notifyAll(); // 多线程中唤醒所有等待线程的功能
obj.wait();     // 多线程中等待功能
obj.toString(); // 把数据转变成字符串
```

### equals 方法

- 一般 equals 和 == 是不一样的，但是在 Object 中两者是一样的。子类一般都要重写这个方法。

### hashCode 方法

- 用于哈希查找，重写了 equals 方法一般都要重写 hashCode 方法，这个方法在一些具有哈希功能的 Collection 中用到。
- **一般必须满足 obj1.equals(obj2)==true 。可以推出 obj1.hashCode()==obj2.hashCode() ，但是hashCode 相等不一定就满足 equals。**不过为了提高效率，应该尽量使上面两个条件接近等价。
  - JDK 1.6、1.7 默认是返回随机数；
  - JDK 1.8 默认是通过和当前线程有关的一个随机数 + 三个确定值，运用 Marsaglia’s xorshift scheme 随机数算法得到的一个随机数。

### wait 方法

- 配合 `synchronized` 使用，wait 方法就是使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。wait() 方法一直等待，直到获得锁或者被中断。wait(long timeout)设定一个超时间隔，如果在规定时间内没有获得锁就返回。

调用该方法后当前**线程进入睡眠状态**，直到以下事件发生。

### notify 方法

- 配合 synchronized 使用，该方法是**等待唤醒的线程**

### notifyAll 方法

- 配合 synchronized 使用，该方法**等待唤醒的所有线程**。

### finalize 方法

- 该方法和垃圾收集器有关系，判断一个对象是否可以被回收的最后一步就是判断是否重写了此方法。

### clone 方法

- 实现对象的浅复制，只有实现了 Cloneable 接口才可以调用该方法，否则抛出CloneNotSupportedException 异常，深拷贝也需要实现 Cloneable，同时其成员变量为引用类型的也需要实现 Cloneable，然后重写 clone 方法。

原文链接：https://blog.csdn.net/csnz123123/article/details/120600792

## **12. == 和 equals 区别是什么?**

**1. 对象类型不同** ：== 是关系运算符，`equals()`是`Object`里的方法，结果都返回布尔值

**2. 比较的对象不同**

1. **equals()：比较两个对象的内容**是否相等。
2. **==：比较两个对象内容和引用地址**是相等。
3. ==：用于比较**引用**和比较**基本数据类型**时具有不同的功能，具体如下：
   1. 基础数据类型：比较的是他们的值是否相等，比如两个int类型的变量，比较的是变量的值是否一样。
   2. 引用数据类型：比较的是引用的地址是否相同，比如说新建了两个User对象，比较的是两个User的地址是否一样。

通过点进去equals源码会发现，其实里面包含了==

```Java
public boolean equals(Object obj) {
    return (this == obj);
}
```

## **18. 为什么重写equals方法必须重写hashcode方法?**

`equals`与`hashcode`间的关系是这样的：

1. 如果两个对象相同，它们`hashCode`值一定要相同；
2. 如果两个对象的`hashCode`相同，两个对象并不一定相同(即用`equals`比较返回`false`)  

## **15. String str = "aaa"与String str = new String("aaa")一样吗? new String("aaa");创建了几个字符串对象?**

```Java
 public class StringTest {
     public static void main(String[] args){
         String s1 = "Hello";
         String s2 = "Hello";
         String s3 = new String("Hello");
         System.out.println("s1和s2 引用地址是否相同："+(s1 == s2));
         System.out.println("s1和s2 值是否相同："+s1.equals(s2));
         System.out.println("s1和s3 引用地址是否相同："+(s1 == s3));
         System.out.println("s1和s3 值是否相同："+s1.equals(s3));
     }
 }
 ---- 
 s1和s2 引用地址是否相同：true
 s1和s2 值是否相同：true
 s1和s3 引用地址是否相同：false
 s1和s3 值是否相同：true
```

- String a = “hello”，是在常量池中创建对象的
- `String s = new String ("2")`; 创建了两个对象，**一个在堆中的****`StringObject`****对象，一个是在常量池中的“****`2`****”对象。**

1. ## **两个new生成的Integer变量的对比**

由于Integer变量实际上是对一个`Integer`对象的引用，所以两个通过`new`生成的`Integer`变量永远是不相等的(因为new生成的是两个对象，其内存地址不同)。

```Java
 Integer i = new Integer(10000);
 Integer j = new Integer(10000) ;
 System.out.print(i == j); //false
```

## **17. throw 和 throws 的区别是什么**

1. throw 是**语句**抛出一个异常；throws 是**方法**抛出一个异常；
2. throws可以单独使用，但throw不能；
3. throw要么和try-catch-finally语句配套使用，要么与throws配套使用。但throws可以单独使用，然后再由处理异常的方法捕获。

## **19. BIO、NIO、AIO的区别?**

- **BIO：同步阻塞**，`一个连接一个线程`，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
- **NIO：同步非阻塞**，`一个请求一个线程`，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
- **AIO：异步非阻塞**，`一个有效请求一个线程`，客户端的 IO 请求都是由 OS 先完成了再通知服务器应用去启动线程进行处理。

**BIO是一个连接一个线程。** **NIO是一个请求一个线程。** **AIO是一个有效请求一个线程。**

**举个例子**

- 同步阻塞：你到饭馆点餐，然后在那等着，啥都干不了，餐没做好，你就必须等着！ 
- 同步非阻塞：你在饭馆点完餐，就去玩儿了。不过玩一会儿，就回饭馆问一声：好了没？ 
- 异步非阻塞：饭馆打电话说，我们知道您的位置，一会给你送过来，安心玩儿就可以了，类似于外卖。