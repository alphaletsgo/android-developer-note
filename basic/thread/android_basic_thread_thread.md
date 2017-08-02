## Java-Thread
#### Java线程

线程化是允许多个活动共存于一个进程的一种工具。


#### 1、线程的生命周期
新建(New)、就绪（Runnable）、运行（Running）、阻塞(Blocked)和死亡(Dead)：

- 新建状态，当程序使用new关键字创建了一个线程之后，该线程就处于新建状态，此时仅由JVM为其分配内存，并初始化其成员变量的值。
- 就绪状态，当线程对象调用了start()方法之后，该线程处于就绪状态。Java虚拟机会为其创建方法调用栈和程序计数器，等待调度运行。
- 运行状态，如果处于就绪状态的线程获得了CPU，开始执行run()方法的线程执行体，则该线程处于运行状态。
- 阻塞状态，当处于运行状态的线程失去所占用资源之后，便进入阻塞状态。
- 死亡状态，当线程的run()方法执行结束后，该线程自然消亡。
参考

#### 2、线程让步——yield

yield()方法和sleep()方法有点相似，它也是Thread类提供的一个静态的方法，它也可以让当前正在执行的线程暂停，让出cpu资源给其他的线程。但是和sleep()方法不同的是，它不会进入到阻塞状态，而是进入到就绪状态。yield()方法只是让当前线程暂停一下，重新进入就绪的线程池中，让系统的线程调度器重新调度器重新调度一次，完全可能出现这样的情况：当某个线程调用yield()方法之后，线程调度器又将其调度出来重新进入到运行状态执行。

实际上，当某个线程调用了yield()方法暂停之后，优先级与当前线程相同，或者优先级比当前线程更高的就绪状态的线程更有可能获得执行的机会，当然，只是有可能，因为我们不可能精确的干涉cpu调度线程。

yield的用法：

```java
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        new MyThread("低级", 1).start();
        new MyThread("中级", 5).start();
        new MyThread("高级", 10).start();
        }
    }

class MyThread extends Thread {
    public MyThread(String name, int pro) {
        super(name);//设置线程的名称
        this.setPriority(pro);//设置优先级
    }

    @Override
    public void run() {
        for (int i = 0; i < 30; i++) {
            System.out.println(this.getName() + "线程第" + i + "次执行！");
            if (i % 5 == 0) Thread.yield();
        }
    }
}
```

关于sleep()方法和yield()方的区别如下：

①、sleep方法暂停当前线程后，会进入阻塞状态，只有当睡眠时间到了，才会转入就绪状态。而yield方法调用后 ，是直接进入就绪状态，所以有可能刚进入就绪状态，又被调度到运行状态。

②、sleep方法声明抛出了InterruptedException，所以调用sleep方法的时候要捕获该异常，或者显示声明抛出该异常。而yield方法则没有声明抛出任务异常。

③、sleep方法比yield方法有更好的可移植性，通常不要依靠yield方法来控制并发线程的执行。

#### 3、线程合并——join

线程的合并的含义就是将几个并行线程的线程合并为一个单线程执行，应用场景是当一个线程必须等待另一个线程执行完毕才能执行时，Thread类提供了join方法来完成这个功能，注意，它不是静态方法。

从上面的方法的列表可以看到，它有3个重载的方法：
```java 
void join()
```
当前线程等该加入该线程后面，等待该线程终止。
```java
void join(long millis)
```
当前线程等待该线程终止的时间最长为 millis 毫秒。 如果在millis时间内，该线程没有执行完，那么当前线程进入就绪状态，重新等待cpu调度。
```java
void join(long millis,int nanos)
```
等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。如果在millis时间内，该线程没有执行完，那么当前线程进入就绪状态，重新等待cpu调度

例子：
```java
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        MyThread thread=new MyThread();
        thread.start();
        thread.join(1);//将主线程加入到子线程后面，不过如果子线程在1毫秒时间内没执行完，则主线程便不再等待它执行完，进入就绪状态，等待cpu调度
        for(int i=0;i<30;i++){
            System.out.println(Thread.currentThread().getName() + "线程第" + i + "次执行！");
        }
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(this.getName() + "线程第" + i + "次执行！");
        }
    }
}
```
在这个例子中，在主线程中调用`thread.join();` 就是将主线程加入到thread子线程后面等待执行。不过有时间限制，为1毫秒。

#### 4、线程的优先级

每个线程执行时都有一个优先级的属性，优先级高的线程可以获得较多的执行机会，而优先级低的线程则获得较少的执行机会。与线程休眠类似，线程的优先级仍然无法保障线程的执行次序。只不过，优先级高的线程获取CPU资源的概率较大，优先级低的也并非没机会执行。

每个线程默认的优先级都与创建它的父线程具有相同的优先级，在默认情况下，main线程具有普通优先级。

Thread类提供了setPriority(int newPriority)和getPriority()方法来设置和返回一个指定线程的优先级，其中setPriority方法的参数是一个整数，范围是1~·0之间，也可以使用Thread类提供的三个静态常量：
```java
MAX_PRIORITY =10

MIN_PRIORITY =1

NORM_PRIORITY =5
```
例子：
```java
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        new MyThread("高级", 10).start();
        new MyThread("低级", 1).start();
        }
    }

    class MyThread extends Thread {
        public MyThread(String name,int pro) {
            super(name);//设置线程的名称
            setPriority(pro);//设置线程的优先级
        }
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                System.out.println(this.getName() + "线程第" + i + "次执行！");
            }
        }
    }
```
从结果可以看到 ，一般情况下，高级线程更显执行完毕。

注意一点：虽然Java提供了10个优先级别，但这些优先级别需要操作系统的支持。不同的操作系统的优先级并不相同，而且也不能很好的和Java的10个优先级别对应。所以我们应该使用MAX_PRIORITY、MIN_PRIORITY和NORM_PRIORITY三个静态常量来设定优先级，这样才能保证程序最好的可移植性。

#### 5、守护线程

守护线程与普通线程写法上基本么啥区别，调用线程对象的方法setDaemon(true)，则可以将其设置为守护线程。

守护线程使用的情况较少，但并非无用，举例来说，JVM的垃圾回收、内存管理等线程都是守护线程。还有就是在做数据库应用时候，使用的数据库连接池，连接池本身也包含着很多后台线程，监控连接个数、超时时间、状态等等。

setDaemon方法的详细说明：

`public final void setDaemon(boolean on)`将该线程标记为守护线程或用户线程。当正在运行的线程都是守护线程时，Java 虚拟机退出。

该方法必须在启动线程前调用。该方法首先调用该线程的checkAccess方法，且不带任何参数。这可能抛出 SecurityException（在当前线程中）。

参数：
```java
on - 如果为 true，则将该线程标记为守护线程。

抛出：

IllegalThreadStateException - 如果该线程处于活动状态。

SecurityException - 如果当前线程无法修改该线程。
```
```java
/** 
* Java线程：线程的调度-守护线程 
*/  
public class Test {  
        public static void main(String[] args) {  
                Thread t1 = new MyCommon();  
                Thread t2 = new Thread(new MyDaemon());  
                t2.setDaemon(true);        //设置为守护线程  

                t2.start();  
                t1.start();  
        }  
}  

class MyCommon extends Thread {  
        public void run() {  
                for (int i = 0; i < 5; i++) {  
                        System.out.println("线程1第" + i + "次执行！");  
                        try {  
                                Thread.sleep(7);  
                        } catch (InterruptedException e) {  
                                e.printStackTrace();  
                        }  
                }  
        }  
}  

class MyDaemon implements Runnable {  
        public void run() {  
                for (long i = 0; i < 9999999L; i++) {  
                        System.out.println("后台线程第" + i + "次执行！");  
                        try {  
                                Thread.sleep(7);  
                        } catch (InterruptedException e) {  
                                e.printStackTrace();  
                        }  
                }  
        }  
}
```
执行结果：
```java
后台线程第0次执行！
线程1第0次执行！
线程1第1次执行！
后台线程第1次执行！
后台线程第2次执行！
线程1第2次执行！
线程1第3次执行！
后台线程第3次执行！
线程1第4次执行！
后台线程第4次执行！
后台线程第5次执行！
后台线程第6次执行！
后台线程第7次执行！
```
从上面的执行结果可以看出：前台线程是保证执行完毕的，后台线程还没有执行完毕就退出了。

实际上：JRE判断程序是否执行结束的标准是所有的前台执线程行完毕了，而不管后台线程的状态，因此，在使用后台县城时候一定要注意这个问题。

守护线程的用途：

守护线程通常用于执行一些后台作业，例如在你的应用程序运行时播放背景音乐，在文字编辑器里做自动语法检查、自动保存等功能。Java的垃圾回收也是一个守护线程。守护线

的好处就是你不需要关心它的结束问题。例如你在你的应用程序运行的时候希望播放背景音乐，如果将这个播放背景音乐的线程设定为非守护线程，那么在用户请求退出的时候，不仅要退出主线程，还要通知播放背景音乐的线程退出；如果设定为守护线程则不需要了。

#### 6、如何结束一个线程

Thread.stop()、Thread.suspend、Thread.resume、Runtime.runFinalizersOnExit这些终止线程运行的方法已经被废弃了，使用它们是极端不安全的！想要安全有效的结束一个线程，可以使用下面的方法。

1、正常执行完run方法，然后结束掉

2、控制循环条件和判断条件的标识符来结束掉线程

比如说run方法这样写：
```java
class MyThread extends Thread {
    int i=0;
    @Override
    public void run() {
        while (true) {
            if(i==10)
                break;
            i++;
            System.out.println(i);

        }
    }
}
```
或者
```java
class MyThread extends Thread {
    int i=0;
    boolean next=true;
    @Override
    public void run() {
        while (next) {
            if(i==10)
                next=false;
            i++;
            System.out.println(i);
        }
    }
}
```
或者
```java
class MyThread extends Thread {
    int i=0;
    @Override
    public void run() {
        while (true) {
            if(i==10)
                return;
            i++;
            System.out.println(i);
        }
    }
}
```
只要保证在一定的情况下，run方法能够执行完毕即可。而不是while(true)的无线循环。

3、使用interrupt结束一个线程。

诚然，使用第2中方法的标识符来结束一个线程，是一个不错的方法，但是如果，该线程是处于sleep、wait、join的状态的时候，while循环就不会执行，那么我们的标识符就无用武之地了，当然也不能再通过它来结束处于这3种状态的线程了。

可以使用interrupt这个巧妙的方式结束掉这个线程。

我们看看sleep、wait、join方法的声明：
```java
public final void wait() throws InterruptedException;
public static native void sleep(long millis) throws InterruptedException;
public final void join() throws InterruptedException;
```
可以看到，这三者有一个共同点，都抛出了一个InterruptedException的异常。

在什么时候会产生这样一个异常呢？

每个Thread都有一个中断状状态，默认为false。可以通过Thread对象的isInterrupted()方法来判断该线程的中断状态。可以通过Thread对象的interrupt()方法将中断状态设置为true。

当一个线程处于sleep、wait、join这三种状态之一的时候，如果此时他的中断状态为true，那么它就会抛出一个InterruptedException的异常，并将中断状态重新设置为false。

看下面的简单的例子：
```java
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        MyThread thread=new MyThread();
        thread.start();
    }
}

class MyThread extends Thread {
    int i=1;
    @Override
    public void run() {
        while (true) {
            System.out.println(i);
            System.out.println(this.isInterrupted());
            try {
                System.out.println("我马上去sleep了");
                Thread.sleep(2000);
                this.interrupt();
            } catch (InterruptedException e) {
                System.out.println("异常捕获了"+this.isInterrupted());
                return;
            }
            i++;
        }
    }
}
```
测试结果：
```java
1
false
我马上去sleep了
2
true
我马上去sleep了
异常捕获了false
```
可以看到，首先执行第一次while循环，在第一次循环中，睡眠2秒，然后将中断状态设置为true。当进入到第二次循环的时候，中断状态就是第一次设置的true，当它再次进入sleep的时候，马上就抛出了InterruptedException异常，然后被我们捕获了。然后中断状态又被重新自动设置为false了（从最后一条输出可以看出来）。

所以，我们可以使用interrupt方法结束一个线程。具体使用如下：
```java
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        MyThread thread=new MyThread();
        thread.start();
        Thread.sleep(3000);
        thread.interrupt();
    }
}

class MyThread extends Thread {
    int i=0;
    @Override
    public void run() {
        while (true) {
            System.out.println(i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println("中断异常被捕获了");
                return;
            }
            i++;
        }
    }
}
```
多测试几次，会发现一般有两种执行结果：
```java
0
1
2
中断异常被捕获了
```
或者
```java
0
1
2
3
中断异常被捕获了
```
这两种结果恰恰说明了:只要一个线程的中断状态一旦为true，只要它进入sleep等状态，或者处于sleep状态，立马回抛出InterruptedException异常。

第一种情况，是当主线程从3秒睡眠状态醒来之后，调用了子线程的interrupt方法，此时子线程正处于sleep状态，立马抛出InterruptedException异常。

第一种情况，是当主线程从3秒睡眠状态醒来之后，调用了子线程的interrupt方法，此时子线程还没有处于sleep状态。然后再第3次while循环的时候，在此进入sleep状态，立马抛出InterruptedException异常。