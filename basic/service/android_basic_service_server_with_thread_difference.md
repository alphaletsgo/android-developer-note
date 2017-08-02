# Service 与 Thread 的区分详解

Thread：Thread 是程序执行的最小单元，它是分配CPU的基本单位。可以用 Thread 来执行一些异步的操作。

Service：Service 是android的一种机制，当它运行的时候如果是Local Service，那么对应的 Service 是运行在主进程的 main 线程上的。如：onCreate，onStart 这些函数在被系统调用的时候都是在主进程的 main 线程上运行的。如果是Remote Service，那么对应的 Service 则是运行在独立进程的 main 线程上。因此请不要把 Service 理解成线程，它跟线程半毛钱的关系都没有！

既然这样，那么我们为什么要用Service呢？其实这跟android的系统机制有关，我们先拿Thread来说。Thread的运行是独立于Activity的，也就是说当一个Activity 被finish之后，如果你没有主动停止Thread或者Thread里的run方法没有执行完毕的话，Thread也会一直执行。因此这里会出现一个问题：当Activity被finish之后，你不再持有该Thread的引用。另一方面，你没有办法在不同的 Activity 中对同一 Thread 进行控制。

举个例子：如果你的 Thread 需要不停地隔一段时间就要连接服务器做某种同步的话，该 Thread 需要在 Activity 没有start的时候也在运行。这个时候当你 start 一个 Activity 就没有办法在该 Activity 里面控制之前创建的 Thread。因此你便需要创建并启动一个 Service ，在 Service 里面创建、运行并控制该 Thread，这样便解决了该问题（因为任何 Activity 都可以控制同一 Service，而系统也只会创建一个对应 Service 的实例）。

因此你可以把Service想象成一种消息服务，而你可以在任何有Context的地方调用`Context.startService`、`Context.stopService`、`Context.bindService`，`Context.unbindService`，来控制它，你也可以在 Service 里注册 BroadcastReceiver，在其他地方通过发送 broadcast 来控制它，当然这些都是 Thread 做不到的。
