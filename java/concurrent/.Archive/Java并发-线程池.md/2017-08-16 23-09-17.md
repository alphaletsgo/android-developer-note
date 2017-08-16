并发在一个app开发中占有很重要的地位，这是也java知识体系中比较重要的一块，Java 1.5版本之前并发对于我们来说实现起来比较麻烦，高兴的是Java 1.5 开始引入 Conccurent 软件包，提供完备的并发能力，对线程池有了更好的支持。其中，Executor 框架是最值得称道的。
Executor框架是指java 1.5中引入的一系列并发库中与executor相关的一些功能类，其中包括线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。并发编程的一种编程方式是把任务拆分为一系列的小任务，即Runnable，然后在提交给一个Executor执行，Executor.execute(Runnalbe) 。Executor在执行时使用内部的线程池完成操作。
![](/imgs/threadpools.png)
这章主要包含线程池和任务两个系列。本文就介绍一下线程池，Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。
## 线程池的好处
- 1、减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
- 2、可以根据系统的承受能力，调整线程池中工作线线程的数目，防止消耗过多的内存。(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)
## 比较重要的类
- ExecutorService：真正的线程池接口
- ScheduledExecutorService：能和Timer/TimerTask类似，解决那些需要任务重复执行的问题
- ThreadPoolExecutor： ExecutorService的默认实现
- ScheduledThreadPoolExecutor：继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现。
## 实例
```java
public class TestFixedThreadPool {
    public static void main(String[] args) {
        //创建一个可重用固定线程数的线程池
        ExecutorService pool = Executors.newFixedThreadPool(2);
        //创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口
        Thread t1 = new MyThread();   //MyThread()是自己随便定义的一个线程，这里就不给出啦。
        Thread t2 = new MyThread();
        Thread t3 = new MyThread();
        Thread t4 = new MyThread();
        Thread t5 = new MyThread();
        //将线程放入池中进行执行
        pool.execute(t1);
        pool.execute(t2);
        pool.execute(t3);
        pool.execute(t4);
        pool.execute(t5);
        //关闭线程池
        pool.shutdown();
    }
}
```
newScheduledThreadPool的实例:
```java
public class TestScheduledThreadPoolExecutor {
    public static void main(String[] args) {
        ScheduledThreadPoolExecutor exec = new ScheduledThreadPoolExecutor(1);
        exec.scheduleAtFixedRate(new Runnable() {//每隔一段时间就触发异常
                      @Override
                      publicvoid run() {
                           //throw new RuntimeException();
                           System.out.println("================");
                      }
                  }, 1000, 5000, TimeUnit.MILLISECONDS);
        exec.scheduleAtFixedRate(new Runnable() {//每隔一段时间打印系统时间，证明两者是互不影响的
                      @Override
                      publicvoid run() {
                           System.out.println(System.nanoTime());
                      }
                  }, 1000, 2000, TimeUnit.MILLISECONDS);
    }
}
```
## ThreadPoolExecutor详解
ThreadPoolExecutor的完整构造方法的签名是：
```java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
```
```
corePoolSize - 池中所保存的线程数，包括空闲线程。
maximumPoolSize-池中允许的最大线程数，>= corePoolSize。
keepAliveTime - 当线程数大于核心时，此为终止多余的空闲线程前等待新任务的最长时间。
unit - keepAliveTime 参数的时间单位。
workQueue - 执行前用于保持任务的队列。此队列仅保持由 execute方法提交的 Runnable任务。
threadFactory - 执行程序创建新线程时使用的工厂。
handler - 当提交任务数超过maxmumPoolSize+workQueue之和时，任务会交给RejectedExecutionHandler来处理。
```
### 重点讲解： 
其中比较容易让人误解的是：corePoolSize，maximumPoolSize，workQueue之间关系。 
- 1）当池子大小小于corePoolSize就新建线程，并处理请求
- 2）当池子大小等于corePoolSize，把请求放入workQueue中，池子里的空闲线程就去从workQueue中取任务并处理
- 3）当workQueue放不下新入的任务时，新建线程入池，并处理请求，如果池子大小撑到了maximumPoolSize就用RejectedExecutionHandler来做拒绝处理
- 4）另外，当池子的线程数大于corePoolSize的时候，多余的线程会等待keepAliveTime长的时间，如果无请求可处理就自行销毁
线程管理机制图示：
![](/imgs/002QxNFVzy75GIBMuB6c4&690.jpg)
ThreadPoolExecutor是Executors类的底层实现，其他几个线程池都是在该类基础上实现的，比如：
```java
public static ExecutorService newFixedThreadPool(int nThreads) {   
    return new ThreadPoolExecutor(nThreads, nThreads,   0L,
     TimeUnit.MILLISECONDS,   new LinkedBlockingQueue<Runnable>());   
}
public static ExecutorService newCachedThreadPool() {   
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,   60L, 
    TimeUnit.SECONDS,new SynchronousQueue<Runnable>());   
}
```
下面就介绍一下ThreadPoolExecutor构造函数中的部分参数
### BlockingQueue workQueue
一共有三种类型的queue。
- 直接提交：工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保存它们。
- 无界队列：使用无界队列（例如，不指定预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。
- 有界队列：当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。
### keepAliveTime
当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间
### RejectedExecutionHandler
可能存在任务过多，线程拒绝接受任务的情况。
RejectedExecutionHandler接口提供了对于拒绝任务的处理的自定方法的机会。在ThreadPoolExecutor中已经默认包含了4种策略：
- CallerRunsPolicy：
线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
   if (!e.isShutdown()) {
       r.run();
   }
}
```

这个策略显然不想放弃执行任务。但是由于池中已经没有任何资源了，那么就直接使用调用该execute的线程本身来执行。
- AbortPolicy：
处理程序遭到拒绝将抛出运行时RejectedExecutionException。
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
   throw new RejectedExecutionException();
}
```
直接抛出异常，丢弃任务
- DiscardPolicy：
不能执行的任务将被删除
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
}
```
这种策略和AbortPolicy几乎一样，也是丢弃任务，只不过他不抛出异常。
- DiscardOldestPolicy：
如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
   if (!e.isShutdown()) {
       e.getQueue().poll();
       e.execute(r);
   }
   }
```
来个热乎的demo
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class CustomExecutor {
    public static final int CORE_POOL_SIZE = 5;
    public static final int MAX_MUM_POOL_SIZE = 9;
    public static final int KEEP_ALIVE_TIME = 5;
    public static final BlockingQueue<Runnable> blockingQueue = new SynchronousQueue<>();
    public static final ThreadFactory threadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable r) {

            return new Thread(r, "CustomExecutor #" + mCount.getAndIncrement());
        }
    };
    public static final RejectedExecutionHandler handler = new RejectedExecutionHandler() {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            System.out.println("队列已满");

        }
    };

    public static ThreadPoolExecutor taskPool = new ThreadPoolExecutor(CORE_POOL_SIZE,MAX_MUM_POOL_SIZE,KEEP_ALIVE_TIME, TimeUnit.SECONDS,blockingQueue,threadFactory,handler);

    public static void main(String args[]){
        for (int i=0;i<20;i++){
            System.out.println(i+";");
            taskPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1_000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+"-"+"wake up");
                }
            });
        }
    }
}
```

### Executors提供的线程池配置方案 
1、构造一个固定线程数目的线程池，配置的corePoolSize与maximumPoolSize大小相同，同时使用了一个无界LinkedBlockingQueue存放阻塞任务，因此多余的任务将存在再阻塞队列，不会由RejectedExecutionHandler处理 
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue());
    }
```
2、构造一个缓冲功能的线程池，配置corePoolSize=0，`maximumPoolSize=Integer.MAX_VALUE`，keepAliveTime=60s,以及一个无容量的阻塞队列 SynchronousQueue，因此任务提交之后，将会创建新的线程执行；线程空闲超过60s将会销毁 
```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue());
    }
```
3、构造一个只支持一个线程的线程池，配置corePoolSize=maximumPoolSize=1，无界阻塞队列LinkedBlockingQueue；保证任务由一个线程串行执行 
```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue()));
    }
```
4、构造有定时功能的线程池，配置corePoolSize，无界延迟阻塞队列DelayedWorkQueue；有意思的是：`maximumPoolSize=Integer.MAX_VALUE`，由于DelayedWorkQueue是无界队列，所以这个值是没有意义的 
```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {
        return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    }

public ScheduledThreadPoolExecutor(int corePoolSize,
                             ThreadFactory threadFactory) {
        super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
              new DelayedWorkQueue(), threadFactory);
    }
```