---
title: Java集合类总结
date: 2016-03-15
author:  KevinWu
categories: Java
tags: 
	- Java 
	- 集合
---


> 这篇文章是对java集合类的个人总结

<!--more-->
﻿

 先来看看java集合类中的框架结构：

- java.util.Collection[interface]

    - java.util.List[interface]

        - java.util.ArrayList[class]

        - java.util.LinkedList[class]

        - java.util.Vector[class]

            - java.util.Stack[class]

    - java.util.set[interface]

        - java.util.HashSet[class]

        - java.util.SortedSet[interface]

            - java.util.TreeSet[class]


- java.util.Map[interface]

    - java.util.SortedMap[interface]

        - java.util.TreeMap[class]

    - java.util.Hashtable[class]

    - java.util.HashMap[class]

        - java.util.LinkedHashMap[class]

        - java.util.WeakHashMap[class]

    - ConcurrentHashMap[class]



# Collection接口

Collection是最基本的集合类接口，一个Collection代表一组Object，即Collection的元素。一些Collection支持相同的元素，而一些不支持，一些支持排序，java不提供直接继承Collection的类，只提供了继承自Collection的子接口的接口入List和Set。


所有集合类都实现了Iterator接口，这是一个遍历集合中的元素的接口，主要包含hasNext()、next()、和remove()三种方法。它的子接口LinkedIterator还在它的基础上加了三种方法：add()、previous()、hasPrevious()。所以只实现了Iterator接口的类，只能往后遍历，被遍历过的元素不会再被遍历到，一般无序的集合都实现了这个接口。而有序的集合一般都实现了LinkedIterator接口，这样可以实现双向遍历，如ArrayList，既可以用next()访问下一个元素，又可以用previous()访问前一个元素。



需要注意一点是，如果要自己去实现一个集合类，就可以实用java定义好的抽象类，这些抽象类已经为我们提供了许多实现，我们只需要根据需求重写一些方法或添加一些方法就行。



根据用途的不同，Collection划分为List和Set。



## List接口

List是继承自Collection的接口。它是有序的，使用这个接口可以精准地控制每个元素插入的位置。可以使用index索引值来访问List中的元素。跟Set不同，List允许有重复的元素。

除了具备Collection接口的Iterator()方法外，List还提供了一个listIterator()方法，返回的是一个ListIterator接口，这个接口与标准Iterator接口相比，多了一些add()之类的方法，允许添加，删除，设定元素，还可以向前，向后遍历。

实现了List接口的常用类有LinkedList，ArrayList，Vector和Stack。



### LinkedList类

LinkedList类实现了List接口，允许插入null元素。可以把它看成双向链表，可以使用LinkedList实现栈（stack）、队列（queue）或双向队列（deque）。

需要注意的是LinkedList没有同步的方法，如果需要在多个线程中访问LinkedList，就需要注意线程同步的问题，或者在创建List时构造一个同步的List，代码如下：

``` java
List<Object> list=Collections.synchronizedList(new LinkedList<>());
```



### ArrayList类

ArrayList可以理解为动态数组。它允许所有的元素，包括null值的元素。ArrayList的初始大小为10，当容量不足时，就会设置成原来的容量的1.5倍加1的新容量，一般确定元素数量的情况下使用ArrayList，否则应该使用LinkedList，因为当容量不足时，都要将原来的元素复制到一个新的数组中，这个操作非常耗时。

同样，ArrayList类也不是同步的，需要注意线程同步问题。



### Vector类

Vector类与ArrayList类非常相似，默认值也是10，当容量不足时，如果制定了incr参数，则增加incr个容量，否则直接在原来容量的基础上加倍。

Vector和ArrayList有别的还有一点，就是Vector是线程同步的。

所以，当一个Iterator被创建且正在被使用时，另一个线程试图改变Vector的元素值时，就会抛出ConcurrentModificationException异常。



#### Stack类

Stack类继承自Vector类，也就是说Vector有的方法它都有，而且在这基础上，实现了先进后出的栈。它提供了5个额外的方法使得一个Vector可以当做堆栈使用，5个方法如下：

- push 进栈

- pop 出栈

- peek 得到栈顶的元素

- empty 测试栈是否为空

- search 检测一个元素在堆栈中的位置



## Set接口

Set和List一样是继承自Collection接口，但与List不同的是，Set是一种不可以包含重复元素的集合。Set类最多只允许有一个null值（有两个null值不就重复了么？～～）。



### HashSet类

这个类实现了Set接口，它底层是基于HashMap实现的，HashSet的底层使用HashMap来保存所有的元素，在Set里面用的只是Map的key，在HashSet里面判断两个元素是否相等，是通过hasCode()和equal()两个方法共同完成的，如果集合的对象中没有重写者两个方法，就会使用object继承来的方法，即比较地址。



### SortedSet接口

SortedSet接口里面的元素一定是有序的。这里的有序不是指插入顺序，而是指根据对象的比较顺序。

对于SortedSet这个接口，目前java仅有一个具体实现的类——TreeSet。



#### TreeSet类

TreeSet类实现Set接口，它是依靠TreeMap实现的，对于TreeSet，需要注意以下几点：

- TreeSet存储对象的时候，可以排序，但是需要指定排序算法。

- Integer可以有默认顺序，String有默认顺序，但是如果是自定义类型，则会出现异常（因为没有默认顺序）

- 如果想把自定义类型存入TreeSet进行排序，那么就需要对该类型实现Comparable接口并重写compareTo()方法，这样在调用add()方法往TreeSet对象添加元素时就会自动调用比较的方法，根据比较的结果使用二叉树的形式存储。



## Map接口

Map接口没有继承Collection接口。也就是说Map和Collection是两种不同类型的集合。Collection可以看做是value（值）的集合，而Map可以看做是key,value（键值对）的集合。

Map有3种集合视图，分别为一组key集合、一组value集合或者一组key-value映射。



### SortedMap接口

保证按照键的升序排列的映射，可以按照键的自然顺序进行排序，或者通过创建有序映射时提供的比较器进行排序。

这个接口的一个比较常用的实现类是TreeMap。



#### TreeMap类

TreeMap类是SortedMap接口的一个具体实现，这个类保证了映射按照升序排序关键字，基于红黑树（自平衡二叉查找树）。根据构造方法的不同，可能按照键值的自然顺序排序，或者按照创建时所提供的比较器进行排序。这个类也是不允许重复值的，和HashMap一样，如果插入重复的元素，后面的元素就会覆盖掉前面的。它的键key不可以为null，但是值可以为null。



### Hashtable类

Hashtable是Map接口的一个具体实现，实现了一个key-value映射的哈希表。Hashtable的key和value都不允许为空。

添加数据时使用put(key,value)，获取一个数据时使用get(key)。

默认容量大小为11，每次扩容，都是将容量变成原来的2倍加1。

Hashtable通过initial capacity和load factor两个参数来调整性能。默认的load factor 0.75较好地实现了时间和空间的均衡。增大load factor可以节省空间但相应的查找时间将会增大，会影响get和put等操作。

Hashtable是线程同步的。



### HashMap类

HashMap也实现了一个key-value映射的哈希表，存储结构和冲突解决与hashtable一致，但HashMap是允许null值的，即key和value的值都允许为null，其中key为null的键值对永远存放在以table[0]为头节点的链表中，当然不一定是存放在头节点table[0]中。

HashMap的默认大小为16，无论设定容量为多少，构造方法都会将实际容量设为不小于指定容量的2的次方的一个数，这样是为了让hash发生碰撞的概率较小，使元素在哈希表中均匀地散列，且这个数的最大值不能超过2的30次方。

每次加入键值对时，都要判断当前已用的槽的数目是否大于等于【容量*load factor】(load factor默认0.75)，如果大于或等于，就进行扩容，每次扩容，容量都编程原来的两倍。



##### 补充：Hashtable和HashMap的区别

它们的主要区别在于：线程安全性、同步以及速度

1. HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的和允许空值null。

2. HashMap是非synchronized的，而Hashtable是synchronized的，这意味者Hashtable是线程安全的，多线程可以共享一个Hashtable，如果没有使用同步，多线程不能共享HashMap。

3. 另一个区别是HashMap的迭代器Iterator是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素）时将会抛出ConcurrentModificationException异常。

4. 由于Hashtable是线程安全的，所以在单线程情况下要比HashMap要慢。

5. HashMap不能保证随着时间的推移Map中的元素次序是不变的。



#### LinkedHashMap类

LinkedHashMap类是HashMap和LinkedList的结合，它的父类是HashMap。它额外定义了一个head为头节点的空的双向循环链表，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的，就是说与键值对的插入顺序保持一致。因为要维护插入顺序，所以性能要低于HashMap，但是在迭代访问元素时会有很好的性能，因为它以链表维护内部顺序。



#### WeakHashMap类

WeakHashMap的用法基本与HashMap相同，区别在于HashMap的key保留对象的强引用，即只要HashMap对象不被销毁，其对象所有key所引用的对象不会被垃圾回收，HashMap也不会自动删除这些key所对应的键值对象。但WeakHashMap的key所引用的对象没有被其他强引用的变量所引用，则这些key所引用的对象可能被回收。WeakHashMap中的每个key对象保存了实际对象的弱引用，当回收了该key所对应的实际对象后，WeakHashMap会自动删除该key对应的键值对。



### ConcurrentHashMap类

ConcurrentHashMap最重要的是引入了Segment的概念，他在自己内部定义了这个Class来管理数据，这个Segment类似与HashMap的定义，ConcurrentHashMap会将对应的读写操作交给Segment。

ConcurrentHashMap默认将Map分成16个Segment，分段锁，有16个写线程执行写，而读的大部分时候都不需要用到锁。只有在size等操作时才需要锁住整个哈希表。















