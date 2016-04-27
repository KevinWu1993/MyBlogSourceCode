---
title: RxJava初步学习总结
date: 2016-03-13
author:  KevinWu
categories: Android
tags: 
	- Java 
	- RxJava
---
** 我们常用的对按钮事件的监听就是观察者模式的一个例子，对于观察者模式，简单的说就是有一个观察者，有个可以被当做观察的对象，这之间通过注册（register）或者订阅（subscribe）的方式进行绑定，当被观察的对象发生改变时，观察者就根据对象的动作做出相应的响应。 **

在OnClickListener事件中，Button作为被观察者，onClickListener作为观察者，通过setOnClickListener订阅，onClick就是被观察者有动作后做出的相应事件。
<!--more-->


由此，引入RxJava中的拓展的观察者模式。

在RxJava中，角色对应如下：

观察者：Observer

被观察者：Observable

订阅：subscribe

响应：onNext()——普通响应
onCompleted()——事件队列完结响应
onError()——事件队列异常响应

（onCompleted()和onError()这两个方法只会调用其中一个，这也很好理解，完结了就肯定没有异常嘛！）



### 创建一个Observer（观察者）

先看一下observer接口的实现：

``` java
Observer<String> observer =new Observer<String>(){

    @override

    public void onNext(String s){

        Log.d(TAG,"Item is :"+s);

    }

    @override

    public void onCompleted(){

        Log.d(TAG,"All things are completed!");

    }    

    @override

    public void onError(Throwable e){

        Log.d(TAG,"Opps! Somethings is error!");

    }

}
```

除了Observer接口，RxJava还有一个Subscriber，翻译过来就是注册者，和观察者一个道理，不过在这里它是一个Observer接口实现的抽象类，只是针对Observer进行了一些扩展，基本的使用方式一样。



在RxJava的订阅（subscribe）过程中，一个Observer也会被转化成Subscriber再使用，但他们也有点区别：

* onStart()：这个是Subscriber新增的方法，会在订阅subscribe刚开始时被调用，可以在这里面做一些准备工作，如对数据进行清零或者重置。需要注意的是这个方法只能在subscribe线程中调用。



* unsubscribe()：这是Subscriber实现另一个接口Subscription的方法，用来取消订阅。这个方法被调用以后，Subscriber将不再继续观察（接收事件）。一般执行这个方法前，应该使用isUnsubscribed()先判断一下状态。unsubscribe()这个方法的使用可以避免内存泄露情况的发生。



### 创建一个Observable（被观察者）

RxJava使用create()方法来创建一个Observable并定义触发规则，下面看看怎么创建一个被观察的对象：

``` java
Observable observable = Observable.create(new Observable.OnSubscribe<String>(){

    @override

    public void call(Subscriber<? super String> subscriber){

        subscriber.onNext("Hello");

        subscriber.onNext("Hi");

        subscriber.onCompleted();

    }

});
```

这里传入一个OnSubscribe对象作为参数，OnSubscribe会被存储在返回的Observable的对象observable中，它的作用相当与一个计划表，当Observable被订阅的时候，OnSubscribe的call方法会自动被调用，事件序列就会依照设定依次触发。这样由被观察者触发了观察者的回调方法，这样就实现了由被观察者向观察者的事件传递。

RxJava还提供了几个快捷创建事件队列的方法，如果要实现上面的功能，还可以使用以下两种方法：

``` java 
Observable observable=Observable.just("Hello","Hi");
```



``` java

String [] info={"Hello","Hi"};
Observable observable=Observable.from(info);
```
### Subscribe（订阅）

订阅后，就将Observer和Observable联结起来了，这个过程就可以正常工作了，订阅的代码如下：

``` java
observable.subscribe(observer);
//或者subscriber，具体看用了哪个
observable.subscribe(subscriber);
```

需要注意的是，这里换成语义是被观察者订阅了观察者，而不是观察者订阅被观察者。



observable.subscribe(subscriber)内部实现中做了三件事：

1. 调用Subscriber.onStart()方法

2. 调用Observable的OnSubscribe.call(Subscriber)方法。在RxJava中，Observable不是一创建就会发送事件，而是在它被订阅的时候，即subscribe()方法被执行的时候

3. 将传入的Subscriber作为Subscription返回，这里可以方便执行unsubscribe()



以上是一个RxJava的学习简单总结，更深入的总结后续继续进行。











