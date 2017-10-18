---
layout: category
title:  "锁、线程、队列"
description: 详解。
categories: java
#categories: [java, lock, thread, queue]
preview: /static/images/common/150/54.jpg
---

----
----


+ 多线程：指的是这个程序（一个进程）运行时产生了不止一个线程
+ 并行与并发：
  - 并行：多个`cpu`实例或者多台机器同时执行一段处理逻辑，是真正的同时。
  - 并发：通过`cpu`调度算法，让用户看上去同时执行，实际上从`cpu`操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用`TPS`或者`QPS`来反应这个系统的处理能力。
+ 线程安全：经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，`cpu`是不是够用即可。反过来，线程不安全就意味着线程的调度顺序会影响最终结果，如不加事务的转账代码：
````java
void transferMoney(User from, User to, float amount){
  to.setMoney(to.getBalance() + amount);
  from.setMoney(from.getBalance() - amount);
}
````
+ 同步：Java中的同步指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。如上面的代码简单加入`@synchronized`关键字。在保证结果准确的同时，提高性能，才是优秀的程序。线程安全的优先级高于性能。

----
----

#### **假如线程`A`和线程`B`使用同一个锁`LOCK`，此时线程`A`首先获取到锁`LOCK.lock()`，并且始终持有不释放。如果此时`B`要去获取锁，有四种方式：**

1. `LOCK.lock()`: 此方式会始终处于等待中，即使调用`B.interrupt()`也不能中断，除非线程`A`调用`LOCK.unlock()`释放锁。
2. `LOCK.lockInterruptibly()`: 此方式会等待，但当调用`B.interrupt()`会被中断等待，并抛出`InterruptedException`异常，否则会与`lock()`一样始终处于等待中，直到线程`A`释放锁。
3. `LOCK.tryLock()`: 该处不会等待，获取不到锁并直接返回`false`，去执行下面的逻辑。
4. `LOCK.tryLock(10, TimeUnit.SECONDS)`：该处会在`10`秒时间内处于等待中，但当调用`B.interrupt()`会被中断等待，并抛出`InterruptedException`。
`10`秒时间内如果线程`A`释放锁，会获取到锁并返回`true`，否则`10`秒过后会获取不到锁并返回`false`，去执行下面的逻辑。

----
----

#### **Java通过Executors提供四种线程池，分别为：**

1. `newCachedThreadPool`创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。 
2. `newFixedThreadPool` 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。 
3. `newScheduledThreadPool` 创建一个定长线程池，支持定时及周期性任务执行。 
4. `newSingleThreadExecutor` 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序`(FIFO, LIFO, 优先级)`执行。

**1) newCachedThreadPool**
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。示例代码如下
````java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
cachedThreadPool.execute(new Runnable() {
    public void run() {
     System.out.println(index);
    }
   });
````
线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

**2) newFixedThreadPool**
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。示例代码如下： 
````java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
fixedThreadPool.execute(new Runnable() {
    public void run() {
     try {
      System.out.println(index);
      Thread.sleep(2000);
     } catch (InterruptedException e) {
      e.printStackTrace();
     }
    }
   });
````
因为线程池大小为`3`，每个任务输出`index`后`sleep 2秒`，所以每两秒打印`3`个数字。 
定长线程池的大小最好根据系统资源进行设置。如`Runtime.getRuntime().availableProcessors()`

**3) newScheduledThreadPool**
创建一个定长线程池，支持定时及周期性任务执行。延迟执行示例代码如下： 
````java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
scheduledThreadPool.schedule(new Runnable() {  
   public void run() {  
    System.out.println("delay 3 seconds");  
   }  
  }, 3, TimeUnit.SECONDS);
````
表示延迟`3`秒执行。  
定期执行示例代码如下： 
````java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
  scheduledThreadPool.scheduleAtFixedRate(new Runnable() {
   public void run() {
    System.out.println("delay 1 seconds, and excute every 3 seconds");
   }
  }, 1, 3, TimeUnit.SECONDS);
````
  表示延迟`1`秒后每`3`秒执行一次。
  
**4) newSingleThreadExecutor**
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序`(FIFO, LIFO, 优先级)`执行。示例代码如下： 
````java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();  
singleThreadExecutor.execute(new Runnable() {  
    public void run() {  
     try {  
      System.out.println(index);  
      Thread.sleep(2000);  
     } catch (InterruptedException e) {  
      e.printStackTrace();  
     }  
    }  
   });
````
结果依次输出，相当于顺序执行各个任务。

**你可以使用JDK自带的监控工具来监控我们创建的线程数量，运行一个不终止的线程，创建指定量的线程，来观察： 
工具目录：C:\Program Files\Java\jdk1.6.0_06\bin\jconsole.exe**

----
----

#### **并发编程-线程池的使用**

**Java中的ThreadPoolExecutor类**  
　　`java.uitl.concurrent.ThreadPoolExecutor`类是线程池中最核心的一个类，因此如果要透彻地了解Java中的线程池，必须先了解这个类。
下面我们来看一下`ThreadPoolExecutor`类的具体实现源码。  
在ThreadPoolExecutor类中提供了四个构造方法：
````java
public class ThreadPoolExecutor extends AbstractExecutorService {
.....
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue);

public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);

public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);

public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
    BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler    handler);
...
}
````
从上面的代码可以得知，`ThreadPoolExecutor`继承了`AbstractExecutorService`类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

下面解释下一下构造器中各个参数的含义：
>*corePoolSize*：核心池的大小， 
这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了`prestartAllCoreThreads()`或者`prestartCoreThread()`方法，
从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建`corePoolSize`个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为`0`，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到`corePoolSize`后，
就会把到达的任务放到缓存队列当中；  
*maximumPoolSize*：线程池最大线程数， 
这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；  
*keepAliveTime*：表示线程没有任务执行时最多保持多久时间会终止。 
默认情况下，只有当线程池中的线程数大于`corePoolSize`时，`keepAliveTime`才会起作用，直到线程池中的线程数不大于`corePoolSize`，即当线程池中的线程数大于`corePoolSize`时，
如果一个线程空闲的时间达到`keepAliveTime`，则会终止，直到线程池中的线程数不超过`corePoolSize`。但是如果调用了`allowCoreThreadTimeOut(boolean)`方法，在线程池中的线程数不大于`corePoolSize`时，`keepAliveTime`参数也会起作用，直到线程池中的线程数为`0`；  
*unit*：参数`keepAliveTime`的时间单位， 有7种取值，在TimeUnit类中有7种静态属性：   
　　`TimeUnit.DAYS;` //天   
　　`TimeUnit.HOURS;` //小时   
　　`TimeUnit.MINUTES;` //分钟   
　　`TimeUnit.SECONDS;` //秒   
　　`TimeUnit.MILLISECONDS;` //毫秒   
　　`TimeUnit.MICROSECONDS;` //微妙   
　　`TimeUnit.NANOSECONDS;` //纳秒  
*workQueue*：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：   
　　`ArrayBlockingQueue;`（有限队列 ）是这样 一种阻塞队列，其中每个 put 必须等待一个 take，反之亦然。同步队列没有任何内部容量。翻译一下：这是一个内部没有任何容量的阻塞队列，任何一次插入操作的元素都要等待相对的删除/读取操作，否则进行插入操作的线程就要一直等待，反之亦然   
　　`LinkedBlockingQueue;` 一个由数组支持的有界阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。
这是一个典型的“有界缓存区”，固定大小的数组在其中保持生产者插入的元素和使用者提取的元素。一旦创建了这样的缓存区，就不能再增加其容量。试图向已满队列中放入元素会导致操作受阻塞；试图从空队列中提取元素将导致类似阻塞。   
　　`SynchronousQueue;` 在ThreadPoolExecutor线程池中常用的等待队列。它可以指定容量也可以不指定容量。由于它具有“无限容量”的特性，所以我还是将它归入了无限队列的范畴（实际上任何无限容量的队列/栈都是有容量的，这个容量就是Integer.MAX_VALUE）。 
LinkedBlockingQueue的实现是基于链表结构，而不是类似ArrayBlockingQueue那样的数组。但实际使用过程中，不需要关心它的内部实现，如果指定了LinkedBlockingQueue的容量大小，那么它反映出来的使用特性就和ArrayBlockingQueue类似了。  
`ArrayBlockingQueue`和`PriorityBlockingQueue`使用较少，一般使用`LinkedBlockingQueue`和`Synchronous`。线程池的排队策略与`BlockingQueue`有关。  
*threadFactory*：线程工厂，主要用来创建线程；  
*handler*：表示当拒绝处理任务时的策略，有以下四种取值：   
　　`ThreadPoolExecutor.AbortPolicy`: 这个处理器，在任务被拒绝后会创建一个`RejectedExecutionException`异常并抛出。这个处理过程也是ThreadPoolExecutor线程池默认的    
　　`ThreadPoolExecutor.DiscardPolicy`：将会默默丢弃这个被拒绝的任务，不会抛出异常，也不会通过其他方式执行这个任务的任何一个方法，更不会出现任何的日志提示。  
　　`ThreadPoolExecutor.DiscardOldestPolicy`：它会检查当前ThreadPoolExecutor线程池的等待队列。并调用队列的poll()方法，将当前处于等待队列列头的等待任务强行取出，然后再试图将当前被拒绝的任务提交到线程池执行（重复此过程）   
　　`ThreadPoolExecutor.CallerRunsPolicy`：这个拒绝处理器，将直接运行这个任务的run方法。但是，请注意并不是在ThreadPoolExecutor线程池中的线程中运行，而是直接调用这个任务实现的run方法。    

　　*实际上查阅这四种ThreadPoolExecutor线程池自带的拒绝处理器实现，您可以发现CallerRunsPolicy、DiscardPolicy、DiscardOldestPolicy处理器针对被拒绝的任务并不是一个很好的处理方式。   
　　CallerRunsPolicy在非线程池以外直接调用任务的run方法，可能会造成线程安全上的问题；DiscardPolicy默默的忽略掉被拒绝任务，也没有输出日志或者提示，开发人员不会知道线程池的处理过程出现了错误；
DiscardOldestPolicy中e.getQueue().poll()的方式好像是科学的，但是如果等待队列出现了容量问题，大多数情况下就是这个线程池的代码出现了BUG。最科学的的还是AbortPolicy提供的处理方式：抛出异常，由开发人员进行处理。*

`ThreadPoolExecutor`继承了`AbstractExecutorService`。  
`AbstractExecutorService`是一个抽象类，它实现了`ExecutorService`接口。 而`ExecutorService`又是继承了`Executor`接口。  
到这里，大家应该明白了`ThreadPoolExecutor`、`AbstractExecutorService`、`ExecutorService`和`Executor`几个之间的关系了。

>`Executor`是一个顶层接口，在它里面只声明了一个方法`execute(Runnable)`，返回值为`void`，参数为`Runnable`类型，从字面意思可以理解，就是用来执行传进去的任务的；     
然后`ExecutorService`接口继承了`Executor`接口，并声明了一些方法：`submit`、`invokeAll`、`invokeAny`以及`shutDown`等；  
抽象类`AbstractExecutorService`实现了`ExecutorService`接口，基本实现了`ExecutorService`中声明的所有方法；   
然后`ThreadPoolExecutor`继承了类`AbstractExecutorService`  

在`ThreadPoolExecutor`类中有几个非常重要的方法：
````java
　　execute()
　　submit()
　　shutdown()
　　shutdownNow()
````
　　`execute()`方法实际上是Executor中声明的方法，在`ThreadPoolExecutor`进行了具体的实现，这个方法是`ThreadPoolExecutor`的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。    
　　`submit()`方法是在`ExecutorService`中声明的方法，在`AbstractExecutorService`就已经有了具体的实现，在`ThreadPoolExecutor`中并没有对其进行重写，这个方法也是用来向线程池提交任务的，
但是它和`execute()`方法不同，它能够返回任务执行的结果，去看`submit()`方法的实现，会发现它实际上还是调用的`execute()`方法，只不过它利用了`Future`来获取任务执行结果。   
　　`shutdown()`和`shutdownNow()`是用来关闭线程池的。   
　　还有很多其他的方法：比如：`getQueue()`、`getPoolSize()`、`getActiveCount()`、`getCompletedTaskCount()`等获取与线程池相关属性的方法，有兴趣的朋友可以自行查阅API。

参考[线程池的使用（ThreadPoolExecutor详解）](http://blog.csdn.net/lipc_/article/details/52025993)

----
----

#### **多线程-队列Queue**


在Java多线程应用中，队列的使用率很高，多数生产消费模型的首选数据结构就是队列(先进先出)。  
*Queue的实现都是用LinkedList。*  一句话：队列是先进先出。相对的，栈是后进先出。  
Java提供的线程安全的`Queue`可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是`BlockingQueue`，非阻塞队列的典型例子是`ConcurrentLinkedQueue`，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。

+ *BlockingQueue，顾名思义，“阻塞队列”：可以提供阻塞功能的队列。*  
  + 首先，看看`BlockingQueue`提供的常用方法：
>add(e) remove() element() 方法不会阻塞线程。当不满足约束条件时，会抛出IllegalStateException 异常。  
例如：当队列被元素填满后，再调用add(e)，则会抛出异常。  
offer(e) poll() peek() 方法即不会阻塞线程，也不会抛出异常。  
例如：当队列被元素填满后，再调用offer(e)，则不会插入元素，函数返回false。  
要想要实现阻塞功能，需要调用put(e) take() 方法。当不满足约束条件时，会阻塞线程。
  + `BlockingQueue`接口的具体实现类：
>`ArrayBlockingQueue` ：一个由数组支持的有界队列。其构造函数必须带一个`int`参数来指明其大小。  
`LinkedBlockingQueue` ：一个由链接节点支持的可选有界队列。实现是线程安全的，实现了先进先出等特性，可以指定容量，也可以不指定，不指定的话，默认最大是`Integer.MAX_VALUE`，
	其中主要用到`put`和`take`方法，`put`方法在队列满的时候会阻塞直到有队列成员被消费，`take`方法在队列空的时候会阻塞，直到有队列成员被放进来。  
`PriorityBlockingQueue` ：一个由优先级堆支持的无界优先级队列。其所含对象的排序不是`FIFO`,而是依据对象的自然排序顺序或者是构造函数的`Comparator`决定的顺序。  
`DelayQueue` ：一个由优先级堆支持的、基于时间的调度队列。  
`SynchronousQueue` ：一个利用 `BlockingQueue` 接口的简单聚集`(rendezvous)`机制。  

+ *ConcurrentLinkedQueue，它是一个无锁的并发线程安全的队列。*  
`ConcurrentLinkedQueue`是`Queue`的一个安全实现．`Queue`中元素按`FIFO`原则进行排序．采用`CAS`操作，来保证元素的一致性。 
>注：`CAS`就是`Compare and Swap`的意思，比较并操作。很多的`cpu`直接支持`CAS`指令。`CAS`是项乐观锁技术，当多个线程尝试使用`CAS`同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。  
`CAS`有`3`个操作数，内存值`V`，旧的预期值`A`，要修改的新值`B`。当且仅当预期值`A`和内存值`V`相同时，将内存值`V`修改为`B`，否则什么都不做。


