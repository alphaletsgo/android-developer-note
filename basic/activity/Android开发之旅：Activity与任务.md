# Android开发之旅：Activity与任务

### 引言

介绍了Android应用程序的进程运行方式：每一个应用程序运行在它自己的Linux进程中。当应用程序中的任何代码需要执行时，Android将启动进程；当它不在需要且系统资源被其他应用程序请求时，Android将关闭进程。而且我们还知道了Android应用程序不像别的应用程序那样（有Main函数入口点），它没有单一的程序入口点，但是它必须要有四个组件中的一个或几个：Activity（Activities） 、服务（Services） 、广播接收者（Broadcast receivers） 、内容提供者（Content providers）。且分别介绍它们的作用，及如何激活和关闭它们、如何在清单文件（AndroidManifest.xml）中声明它们及Intent过滤器。

在简单回顾之后，本篇还是继续介绍Android应用程序原理及术语——Activity与任务（Activities and Tasks）。

- 1、Activity与任务概述
- 2、亲和度（Affinity）
- 3、启动模式（Launch modes）
- 4、清除栈（Clearing the stack）
- 5、启动任务（Starting tasks）

### 1、Activity与任务概述

如前所述，一个activity能启动另一个activity，包括定义在别的应用程序中的activity。举例说明，假设你想让用户显示某地的街道地图。而且已经有了一个activity能做这个事情（这个activity可以是另一个程序中的），因此你的activity要做的就是将请求信息放进一个Intent对象，然后将它传给`startActivity()`。地图查看器就启动并显示出地图。当用户点击返回按钮之后，你的activity就会重新出现在屏幕上。

对用户来说，这个地图查看器就好像是你的应用程序的activity一样，虽然它定义在其他的应用程序中且运行在那个应用程序的进程中。Android将这些activity保持在同一个任务（task）中以维持用户的体验。**简单地讲，任务是一个Activity的集合，它使用栈的方式来管理其中的Activity，这个栈又被称为返回栈(back stack)，栈中Activity的顺序就是按照它们被打开的顺序依次存放的**。栈底的Activity（根Activity）是起始Activity——一般来讲，它是用户在应用程序启动器（也称应用程序列表，下同）中选择的一个Activity。栈顶的Activity是正在运行的Activity——它关注用户的行为（操作）。当一个Activity启动另一个Activity，新的Activity动被压入栈顶，变为正在运行的Activity。前面那个Activity保存在栈中。当用户点击返回按钮时，当前Activity从栈顶中弹出，且前面那个Activity恢复成为正在运行的Activity。

栈中包含对象，因此如果一个Activity启动了多个实例——例如多个地图查看器，则栈对每个实例有一个独立的入口。（可以这样理解：假设有四个Activity以这样的顺序排在栈中——A-B-C-D，现在又有一个C的实例，则栈变成A-B-C-D-C，这两个C的实例是独立的。）栈中的Activity从不会被重新排列，只会被压入、弹出。这点很好理解，因为Activity的调用顺序是固定的。

任务是一栈的Activity，而不是清单文件中声明的某个类或元素，因此无法独立于它的Activity为任务赋值。整个任务的值是在栈底Activity（根Activity）设置的。例如，下节将讨论的“任务亲和度”，亲和度信息就是从任务的根Activity中获取的。

一个任务的所有Activity作为一个整体运行。整个任务（整个Activity栈）可置于前台或发送到后台。例如，假设当前任务有四个Activity在栈中——三个Activity在当前Activity下面。用户按下HOME键，切换到程序启动器，并选择一个新的应用程序（实际上是一个新的任务）。当前任务进入后台，新任务的根Activity将显示。接着，过了一会，用户回到主屏幕并再次选择之前的应用程序（之前的任务）。那个任务栈中的所有四个Activity都变为前台运行。当用户按下返回键时，不是离开当前任务回到之前任务的根Activity。相反，栈顶的Activity被移除且栈中的下一个Activity将显示。

上面所描述的是Activity和任务的默认行为，这些行为是可以通过方法来改变的。Activity与任务之间的联系及任务中Activity的行为，是由启动Activity的Intent对象的标志（flags）和清单文件中Activity`<activity>`元素的属性共同决定的。

在这方面，主要的Intent标志有：
```xml
FLAG_ACTIVITY_NEW_TASK
FLAG_ACTIVITY_CLEAR_TOP
FLAG_ACTIVITY_RESET_TASK_IF_NEEDED
FLAG_ACTIVITY_SINGLE_TOP
```
主要的`<activity>`属性有：
```xml
taskAffinity
launchMode
allowTaskReparenting
clearTaskOnLaunch
alwaysRetainTaskState
finishOnTaskLaunch
```
接下来的小节将讨论这些标志和属性的作用，他们怎么交互，及使用的注意事项。

### 2、亲和度（Affinity）

affinity可以用于指定一个Activity更加愿意依附于哪一个任务，在默认情况下，同一个应用程序中的所有Activity都具有相同的affinity，所以，这些Activity都更加倾向于运行在相同的任务当中。当然了，你也可以去改变每个Activity的affinity值，通过<activity>元素的taskAffinity属性就可以实现了。

#### affinity主要有以下两种应用场景：

- 当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了一个FLAG_ACTIVITY_NEW_TASK flag的话(或者该Activity在manifest文件中声明的启动模式是"singleTask")，系统就会尝试为这个Activity单独创建一个任务。但是规则并不是只有这么简单，系统会去检测要启动的这个Activity的affinity和当前任务的affinity是否相同，如果相同的话就会把它放入到现有任务当中，如果不同则会去创建一个新的任务。而同一个程序中所有Activity的affinity默认都是相同的，这也是前面为什么说，同一个应用程序中即使声明成"singleTask"，也不会为这个Activity再去创建一个新的任务了。

- 当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在任务的能力。具体点来说，就是一个Activity现在是处于某个任务当中的，但是它与另外一个任务具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。
那还是举一个形象点的例子吧，比如有一个天气预报程序，它有一个Activity是专门用于显示天气信息的，这个Activity和该天气预报程序的所有其它Activity具体相同的affinity值，并且还将allowTaskReparenting属性设置成true了。这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，这个Activity又会被转移到天气预报程序的任务当中，并显示出来，因为它们拥有相同的affinity值，并且将allowTaskReparenting属性设置成了true。

### 3、启动模式（Launch modes）

启动模式允许你去定义如何将一个Activity的实例和当前的任务进行关联，你可以通过以下两种不同的方式来定义启动模式：

#### 1.使用manifest文件
当你在manifest文件中声明一个Activity的时候，你可以指定这个Activity在启动的时候该如何与任务进行关联。

#### 2.使用Intent flag
当你调用startActivity()方法时，你可以在Intent中加入一个flag，从而指定新启动的Activity该如何与当前任务进行关联。

也就是说，如果Activity A启动了Activity B，Activity B可以定义自己该如何与当前任务进行关联，而Activity A也可以要求Activity B该如何与当前任务进行关联。如果Activity B在manifest中已经定义了该如何与任务进行关联，而Activity A同时也在Intent中要求了Activity B该怎么样与当前任务进行关联，那么此时Intent中的定义将覆盖manifest中的定义。

需要注意的是，有些启动模式在manifest中可以指定，但在Intent中是指定不了的。同样，也有些启动模式在Intent中可以指定，但在manifest中是指定不了的，下面我们就来具体讨论一下。

#### 使用manifest文件

当在manifest文件中定义Activity的时候，你可以通过<activity>元素的launchMode属性来指定这个Activity应该如何与任务进行关联。launchMode属性一共有以下四种可选参数：

**"standard"(默认启动模式)**

standard是默认的启动模式，即如果不指定launchMode属性，则自动就会使用这种启动模式。这种启动模式表示每次启动该Activity时系统都会为创建一个新的实例，并且总会把它放入到当前的任务当中。声明成这种启动模式的Activity可以被实例化多次，一个任务当中也可以包含多个这种Activity的实例。

**"singleTop"**

这种启动模式表示，如果要启动的这个Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么系统就不会再去创建一个该Activity的实例，而是调用栈顶Activity的onNewIntent()方法。声明成这种启动模式的Activity也可以被实例化多次，一个任务当中也可以包含多个这种Activity的实例。

举个例子来讲，一个任务的返回栈中有A、B、C、D四个Activity，其中A在最底端，D在最顶端。这个时候如果我们要求再启动一次D，并且D的启动模式是"standard"，那么系统就会再创建一个D的实例放入到返回栈中，此时栈内元素为：A-B-C-D-D。而如果D的启动模式是"singleTop"的话，由于D已经是在栈顶了，那么系统就不会再创建一个D的实例，而是直接调用D Activity的onNewIntent()方法，此时栈内元素仍然为：A-B-C-D。

**"singleTask"**

这种启动模式表示，系统会创建一个新的任务，并将启动的Activity放入这个新任务的栈底位置。但是，如果现有任务当中已经存在一个该Activity的实例了，那么系统就不会再创建一次它的实例，而是会直接调用它的onNewIntent()方法。声明成这种启动模式的Activity，在同一个任务当中只会存在一个实例。注意这里我们所说的启动Activity，都指的是启动其它应用程序中的Activity，因为"singleTask"模式在默认情况下只有启动其它程序的Activity才会创建一个新的任务，启动自己程序中的Activity还是会使用相同的任务，具体原因会在下面 处理affinity 部分进行解释。

**"singleInstance"**

这种启动模式和"singleTask"有点相似，只不过系统不会向声明成"singleInstance"的Activity所在的任务当中再添加其它Activity。也就是说，这种Activity所在的任务中始终只会有一个Activity，通过这个Activity再打开的其它Activity也会被放入到别的任务当中。

再举一个例子，Android系统内置的浏览器程序声明自己浏览网页的Activity始终应该在一个独立的任务当中打开，也就是通过在<activity>元素中设置"singleTask"启动模式来实现的。这意味着，当你的程序准备去打开Android内置浏览器的时候，新打开的Activity并不会放入到你当前的任务中，而是会启动一个新的任务。而如果浏览器程序在后台已经存在一个任务了，则会把这个任务切换到前台。

其实不管是Activity在一个新任务当中启动，还是在当前任务中启动，返回键永远都会把我们带回到之前的一个Activity中的。但是有一种情况是比较特殊的，就是如果Activity指定了启动模式是"singleTask"，并且启动的是另外一个应用程序中的Activity，这个时候当发现该Activity正好处于一个后台任务当中的话，就会直接将这整个后台任务一起切换到前台。此时按下返回键会优先将目前最前台的任务(刚刚从后台切换到最前台)进行回退，下图比较形象地展示了这种情况：

![png](/imgs/diagram_backstack_singletask_multiactivity.png)

#### 使用Intent flags

除了使用manifest文件之外，你也可以在调用startActivity()方法的时候，为Intent加入一个flag来改变Activity与任务的关联方式，下面我们来一一讲解一下每种flag的作用：

**FLAG_ACTIVITY_NEW_TASK**

设置了这个flag，新启动Activity就会被放置到一个新的任务当中(与"singleTask"有点类似，但不完全一样)，当然这里讨论的仍然还是启动其它程序中的Activity。这个flag的作用通常是模拟一种Launcher的行为，即列出一推可以启动的东西，但启动的每一个Activity都是在运行在自己独立的任务当中的。

**FLAG_ACTIVITY_SINGLE_TOP**

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么就不会再次创建这个Activity的实例，而是直接调用它的onNewIntent()方法。这种flag和在launchMode中指定"singleTop"模式所实现的效果是一样的。

**FLAG_ACTIVITY_CLEAR_TOP**

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，就不会再次创建这个Activity的实例，而是会把这个Activity之上的所有Activity全部关闭掉。比如说，一个任务当中有A、B、C、D四个Activity，然后D调用了startActivity()方法来启动B，并将flag指定成FLAG_ACTIVITY_CLEAR_TOP，那么此时C和D就会被关闭掉，现在返回栈中就只剩下A和B了。

那么此时Activity B会接收到这个启动它的Intent，你可以决定是让Activity B调用onNewIntent()方法(不会创建新的实例)，还是将Activity B销毁掉并重新创建实例。如果Activity B没有在manifest中指定任何启动模式(也就是"standard"模式)，并且Intent中也没有加入一个FLAG_ACTIVITY_SINGLE_TOP flag，那么此时Activity B就会销毁掉，然后重新创建实例。而如果Activity B在manifest中指定了任何一种启动模式，或者是在Intent中加入了一个FLAG_ACTIVITY_SINGLE_TOP flag，那么就会调用Activity B的onNewIntent()方法。

FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_NEW_TASK结合在一起使用也会有比较好的效果，比如可以将一个后台运行的任务切换到前台，并把目标Activity之上的其它Activity全部关闭掉。这个功能在某些情况下非常有用，比如说从通知栏启动Activity的时候。

### 4、清除栈（Clearing the stack）

如果用户离开一个任务很长时间，系统将会清除根Activity之外的Activity。当用户再次返回到这个任务时，像用户离开时一样，仅显示初始的Activity。这个想法是，一段时间后，用户可能已经放弃之前做的东西，及返回任务做新的事情。这是默认情况，有些Activity属性可以用来控制和改变这个行为。

#### alwaysRetainTaskState属性

如果在任务的根Activity中这个属性被设置为"true"，刚才描述的默认行为将不会发生。任务将保留所有的Activity在它的栈中，甚至是离开很长一段时间。

#### clearTaskOnLaunch属性

如果在任务的根Activity中这个属性被设置为"true"，只有用户离开就清除根Activity之外的Activity。换句话说，它与alwaysRetainTaskState截然相反。用户总是返回到任务的初始状态，甚至是只离开一会。

#### finishOnTaskLaunch属性

这个属性类似于clearTaskOnLaunch，但是它作用于单个Activity，而不是整个任务。而且它能移除任何Activity，包括根Activity。当它被设置为"true"，任务本次会话的Activity的部分还存在，如果用户离开并返回到任务时，它将不再存在。

### 5、启动任务（Starting tasks）

通过给定Activity一个意图过滤器`"android.intent.action.MAIN"`作为指定行为（action）和`"android.intent.category.LAUNCHER"`指定种类（category），Activity就被设置为任务的入口点了。

如果您希望用户离开Activity后就不能再回到这个Activity，可以将`<activity>`元素的finishOnTaskLaunch属性设置为"true"。