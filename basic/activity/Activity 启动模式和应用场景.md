# Activity 启动模式和应用场景

### 模式简介
Android Activity有四种启动模式：
- standard
- singleTop
- singleTask
- singleInstance

**standard:** 也是Activity默认的启动方式，若将activity模式设置为standard，那么启动该activity时不管任务栈是否存在都会创建新的实例对象。

**singleTop:** 表示若该activity处于任务栈的栈顶时不会实例化新的对象，但会掉`onNewIntent()`方法，若该任务栈中已经存在但不在栈顶，那么则跟standard模式一样。

**singleTask:** [参考](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)

**singleInstance:** [参考](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)


### 使用场景
**standard:** 适合多个实例存在的情况，比如，发邮件页面。

**singleTop:** 适合接收通知内容显示页面。例如，某些应用会为用户推送一些消息通知，当用户从任务栏中进入查看消息内容界面时，如果设置为singleTop时，这样每次行为都使用同一个实例，用户点击返回时不会存在多个消息页面的情况。

**singleTask:** 适合使用在一个程序的主界面。

**singleInstance:** 使用较少，比如一些launchAPP可能会使用到。

## 总结
这里总结一下这篇[文章](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)关于对activity启动模式的介绍。

首先是默认的standard，关于该模式的表现存在两种不同的情况（针对不同应用间调用），在5.0之前，一个应用启动另一个应用的activity（默认standard）会在调用者任务栈中去实例化activity对象。5.0之后会启动一个新的任务栈去实例化。

其次是singleTask，这种模式实际表现与官方文档介绍的有一点差异。默认情况下，在同一应用内启动一个模式为singleTask的activity时，若该应用栈里存在则移除栈中该实例之上的所有实例以确保该实例处于栈顶，若不存则在该栈中创建新的实例，在这里与官方文档中会创建一个新的任务栈不太一样，若要表现与官方介绍一样的话需要设置`android:taskAffinity=""`即可。在不同应用之间调用：若存在该实例的任务栈并且该实例处于栈顶时，则使用该任务栈里地实例对象，如果存在该实例的任务栈但不是处于栈顶，则移除该实例之上的所有实例。若不存在该实例的任务栈，则创建新的任务栈并实例化对象。

最后是singleInstance, 这个也是跟官方文档的介绍有所不同，解决方案跟singleInstance一样。

## 参考文献
[Google官方文档](http://developer.android.com/guide/components/tasks-and-back-stack.html)