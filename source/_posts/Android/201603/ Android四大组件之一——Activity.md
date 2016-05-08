title: Android四大组件之一——Activity
date: 2016-03-22
author:  KevinWu
categories: Android
tags: 
	- Java 
	- Android
	- 加载模式
	- Activity
	- 启动模式
---
> Activity是Android的四大组件之一，这里将梳理一下有关Activity的东西



## Activity的四种状态

**Active状态：**在这种状态的Activity，必须是在栈顶的，此时它是可视的，并且可以获取焦点和用户的输入，Android系统将会保证在这种状态的Activity有充分的资源，当资源不足时就会试图杀死其它Activity来提供足够的资源，当激活另一个Activity后，当前Active状态的Activity将会进入Paused状态。
<!--more-->
**Paused状态：**当一个透明或者非全屏的Activity被激活时，前一个处于Active状态的Activity将会进入Paused状态，处于这种状态的Activity依然处于活动状态，只是不再能获取焦点和用户输入，如果当前被激活的Activity的资源足够的情况下，Android不会主动杀死处于Paused状态的Activity，当一个Activity变得完全不可见时，它将会进入Stoped状态。

**Stoped状态：**当一个Activity变得完全不可见，就代表它此时进入了Stoped状态。虽然此时该Activity被停止了，但内存中仍保存这这个Activity的所有信息，但当Android出现内存资源不足时，处于这种状态的Activity是最有可能被杀死的。当一个Activity退出或关闭了，就会进入Killed状态。

**Killed状态：**当一个Activity被退出了或者关闭了，就进入了这种状态，处于这种状态的Activity，在需要时可以重新启用它。



## Activity生命周期

*了解一个组件，生命周期这个一定要了解，才能更好地把握在什么时候该做什么事。*

说到生命周期，必然少不了这张图：



#### 简单说明一下这张图：

![](http://i12.tietuku.cn/2bf48e46914376f0.png)
- 当一个Activity启动后，系统会先调用Activity中的onCreate()方法，这个方法也是见得最多的。

- 执行完onCreate()方法后，系统会依次调用onStart()、onResume()方法，调用完这个方法后，一个Activity就算正式启动完成了，此时Activity显示在前台，也就是处于前面所说的四种状态中的Active状态。

- 当另一个Activity被激活，并占据前台时，当前处于Active状态的Activity将执行onPause()状态，此时也就是前面所说的四种状态中的Paused状态。此时如果系统资源不足，需要资源，很可能处于这个状态的Activity将被杀死，就进入了Killed状态，又或者，处于这个状态的Activity又重新回到前台，那么将重新调用onResume()方法，Activity将重新进入前台显示。

- 当一个Activity不再可见后，即执行该Activity的onStop()方法，执行该方法说明该Activity进入了前面四种状态中的Stoped状态，此时也跟Paused状态一样，有两种可能，只是这里在回到前台后，执行的不是onResume()方法，而是先执行onRestart()方法。然后执行onStart()方法，再执行onResume()方法。（我个人把这个理解为重新启动的过程，不知道这么理解有没有问题）

- 当用户退出Activity后，如果处于任何状态的Activity都会按照图示顺序执行下来，执行onStop()方法后，就会执行onDestory()方法，执行完onDestory()方法后，这个Activity将被正式关闭了。



#### 下面引用网上对这几种方法的介绍，来说明下这几种方法：

- onCreate：当Activity第一次启动的时候，触发这个方法，可以在此时完成Activity的初始化工作。onCreate方法有一个参数，这个参数可以为空，也可以是之前调用onSaveInstanceState()方法保存的状态信息。

- onStart：该方法的触发表示所属活动将被展现给用户。

- onResume：当一个活动和用户发生交互时，就会触发该方法。

- onPause：当一个正在前台运行的活动因为其他活动需要前台运行而转入后台运行的时候，触发该方法。这时候需要将活动的状态持久化，比如正在编辑的数据库记录等。

- onStop：当一个活动不需要展示给用户的时候，就会触发该方法。如果内存紧张，系统会直接结束这个活动，而不会触发onStop方法。所以保存状态信息是应该在onPause时做，而不是onStop时候做。活动如果没有在前台运行，都将被停止或者Linux管理进程为了给新的活动预留足够的存储空间而随时结束这些活动，因此对于开发者来说，在设计应用程序的时候，必须时刻牢记这一原则。在一些情况下，onPause方法或许是活动触发的最后的方法，因此开发者需要在这个时候保存需要保存的信息。

- onRestart：当处于停止状态的活动需要再次展现给用户的时候，触发该方法。

- onDestory：当活动销毁的时候，触发该方法。和onStop方法一样，如果内存紧张，系统会直接结束这个活动而不会触发这个方法。



另：

**onSaveInstanceState：**系统调用该方法，允许活动保存之前的状态，比如说在一串字符串中的光标所处的位置等。

通常情况下，开发者不需要重写覆盖该方法，在默认实现中，已经提供了自动保存活动所设计的用户界面组件的所有状态信息。



## Activity的四种加载模式

Activity提供了四种加载模式，分别为：standard、singleTop、singleTask、singleInstance模式。

这四种模式都是在AndroidManifest.xml文件中通过android:lanchMode来指定的，在介绍这四种启动模式之前，有必要先来了解一个概念——**Activity栈**



### Activity栈

Android是通过栈来保存Activity的，栈底的元素就是整个任务的发起者。

当一个应用启动时，如果当前没有该应用的任务栈，就会创建一个任务栈，有了任务栈后，这个应用所启动的Activity都将在这个任务栈中被管理。

有一点需要注意的，一个任务栈中Activity可能来自不同应用，同一个应用的Activity也可能不再一个任务栈中。

一般情况下，当一个任务栈中的一个Activity启动了另一个Activity的时候，新启动的Activity就会进入任务栈顶端，并处于Active状态，而启动它的Activity就被顶下去一个位置，处于Stoped状态（不可见的情况下），当当前Activity执行完后，即用户按下返回键或者调用了finish()方法后，当前Activity出栈，启动它的Activity就宠幸回到Active状态。

这就是基于栈的后进先出的任务栈。

### standard模式

这是默认的加载模式，这种加载模式每次都会创建一个新的Activity，每创建一次，新的Activity就会覆盖在原有的Activity上。

### singleTop模式

我个人理解就相当与栈顶单实例模式，设置了这种模式的Activity，在启动时会先判断该Activity是否处于栈顶，如果处于栈顶就不会新建Activity实例，而是直接重新启动在栈顶的这个Activity，并调用onNewIntent()方法。

### singleTask模式

singleTask模式和singleTop模式类似，不过与singleTop检测栈顶元素是否是需要启动的Activity不同的是，singleTask是检测整个Activity栈中是否存在需要启动的Activity，如果存在，就把该Activity顶到栈顶，在该Activity上的Activity都会出栈销毁。

上面是对处于同一个应用中启动的singleTask模式的Activity的处理。

对于在其他应用中启动singleTask模式的Activity，如果后台任务栈不存在要启动的Activity，那么将会创建一个新的任务栈，如果后台任务栈中存在要启动的Activity，该后台任务栈就会切换到前台。

### singleInstance

singleInstance可以简单理解为singleTask的加强版，这个加载模式具备singleTask的所有特性，但在任务栈的处理上加强了，处于这种模式的Activity，每一个Activity都会单独处于一个任务栈中，而且那个任务栈中智能存在单个Activity。



