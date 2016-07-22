---
title: GC-垃圾回收机制
date: 2016-07-19
author:  KevinWu
categories: Java
tags: 
	- Java 
	- GC
	- JVM
---
这是一篇我之前在ImportNew公众号看到的文章，觉得写得很好，虽然我现在也不确定是否会每次都如作者所说会Stop-the-world（抱着疑问），但看完后还会时不时想起它了，如今又一次看到它，毅然收到自己博客，如果原作者看到不服请联系我，我马上把它删了谢谢。
再次说明，这是一篇转载文章，原文地址为： http://blog.csdn.net/wsl211511/article/details/51099051
<!--more-->
# 意义
在C++中，对象所占的内存在程序结束运行之前一直被占用，在明确释放之前不能分配给其它对象；而在Java中，当没有对象引用指向原先分配给某个对象的内存时，该内存便成为垃圾。JVM的一个系统级线程会自动释放该内存块。垃圾收集意味着程序不再需要的对象是"无用信息"，这些信息将被丢弃。当一个对象不再被引用的时候，内存回收它占领的空间，以便空间被后来的新对象使用。事实上，除了释放没用的对象，垃圾收集也可以清除内存记录碎片。由于创建对象和垃圾收集器释放丢弃对象所占的内存空间，内存会出现碎片。碎片是分配给对象的内存块之间的空闲内存洞。碎片整理将所占用的堆内存移到堆的一端，JVM将整理出的内存分配给新的对象。

垃圾收集能自动释放内存空间，减轻编程的负担。这使Java 虚拟机具有一些优点。首先，它能使编程效率提高。在没有垃圾收集机制的时候，可能要花许多时间来解决一个难懂的存储器问题。在用Java语言编程的时候，靠垃圾收集机制可大大缩短时间。其次是它保护程序的完整性，垃圾收集是Java语言安全性策略的一个重要部份。

垃圾收集的一个潜在的缺点是它的开销影响程序性能。Java虚拟机必须追踪运行程序中有用的对象，而且最终释放没用的对象。这一个过程需要花费处理器的时间。其次垃圾收集算法的不完备性，早先采用的某些垃圾收集算法就不能保证100%收集到所有的废弃内存。当然随着垃圾收集算法的不断改进以及软硬件运行效率的不断提升，这些问题都可以迎刃而解。

一般来说，Java开发人员可以不重视JVM中堆内存的分配和垃圾处理收集，但是，充分理解Java的这一特性可以让我们更有效地利用资源。同时要注意finalize()方法是Java的缺省机制，有时为确保对象资源的明确释放，可以编写自己的finalize方法。

# 原理
关于垃圾收集器，在学习GC前，你应该知道一个技术名词:这个词是“stop-the-world。“无论你选择哪种GC算法，Stop-the-world都会发生。Stop-the-world意味着JVM停止应用程序，而去进行垃圾回收。当stop-the-world发生时，除了进行垃圾回收的线程，其他所有线程都将停止运行。被中断的任务将在GC任务完成后恢复执行。GC调优往往意味着减少stop-the-world的时间。

## 分代垃圾收集
在Java代码中，Java语言没有显式的提供分配内存和删除内存的方法。一些开发人员将引用对象设置为null或者调用System.gc()来释放内存。将引用对象设置为null没有什么大问题，但是调用system.gc()方法会大大的影响系统性能，绝对不能这个干。(谢天谢地，我还没看到任何NHN开发者调用这个方法。)

在Java中，由于开发人员没有在代码中显式删除内存，所以垃圾收集器会去发现不需要（垃圾）的对象,然后删除它们，释放内存。这款垃圾收集器是基于以下两个假设而创建的。（称他们为前提条件更好，而不是假设。）

绝大多数对象在短时间内变得不可达，只有少量年老对象引用年轻对象。这些假设被称为“弱代假说”。为了发挥这一假设的优势，在HotSpot虚拟机中，物理的将内存分为两个—年轻代(young generation)和老年代(old generation)。

年轻代：新创建的对象都存放在这里。因为大多数对象很快变得不可达，所以大多数对象在年轻代中创建，然后消失。当对象从这块内存区域消失时，我们说发生了一次“minor GC”。

老年代：没有变得不可达，存活下来的年轻代对象被复制到这里。这块内存区域一般大于年轻代。因为它更大的规模，GC发生的次数比在年轻代的少。对象从老年代消失时，我们说“major GC”（或“full GC”）发生了。

我们看一下这幅图：
![GC区 & 数据流](http://img.blog.csdn.net/20160408192232373?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

上图中的永久代(permanent generation)也称为“方法区(method area)”，他存储class对象和字符串常量。所以这块内存区域绝对不是永久的存放从老年代存活下来的对象的。在这块内存中有可能发生垃圾回收。发生在这里垃圾回收也被称为major GC。

一些人可能想知道:一个老年代的对象需要引用年轻代的对象，该怎么办？

为了解决这些问题，老年代中有一个被称为“卡表(card table)”的东西，它是一个512 byte大小的块。每当老年代的对象引用年轻代对象时，这种引用会被记录在这张表格中。当垃圾回收发生在年轻代时，只需对这张表进行搜索以确定是否需要进行垃圾回收，而不是检查老年代中的所有对象引用。这张表格用一个叫做“写闸(write barrier)”的东西进行管理。“写闸”是一种装置，对minor GC有更好性能。虽然因为这种机制，会产生一些时间性能开销，但降低了整体的GC时间。
![](http://img.blog.csdn.net/20160408192305019?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 年轻代组成部分
为了理解GC，我们学习一下年轻代，对象第一次创建发生在这块内存区域。年轻代分为3块，Eden区和2个Survivor区。

年轻代总共有3块空间，其中2块为Survivor区。各个空间的执行顺序如下：

绝大多数新创建的对象分配在Eden区。

在Eden区发生一次GC后，存活的对象移到其中一个Survivor区。

在Eden区发生一次GC后，对象是存放到Survivor区，这个Survivor区已经存在其他存活的对象。

一旦一个Survivor区已满，存活的对象移动到另外一个Survivor区。然后之前那个空间已满Survivor区将置为空，没有任何数据。

经过重复多次这样的步骤后依旧存活的对象将被移到老年代。

通过检查这些步骤，如你看到的样子，其中一个Survivor区必须保持空。如果数据存在于两个Survivor区，或两个都没使用，你可以将这个情况作为系统错误的一个标志。

经过多次minor GC,数据被转移到老年代过程如下面的图表所示:
![GC前和GC后](http://img.blog.csdn.net/20160408192401047?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

请注意，在HotSpot虚拟机中，使用两种技术加快内存的分配。一个被称为“指针碰撞(bump-the-pointer)”，另外一个被称为“TLABs（线程本地分配缓冲）”。

指针碰撞技术跟踪分配给Eden区上最新的对象。该对象将位于Eden 区的顶部。如果之后有一个对象被创建，只需检查Eden区是否有足够大的空间存放该对象。如果空间够用，它将被放置在Eden区，存放在空间的顶部。因此，在创建新对象时，只需检查最后被添加对象，看是否还有更多的内存空间允许分配。然而，如果考虑多线程的环境，则是另外一种情况。为了实现多线程环境下，在Eden 区线程安全的去创建保存对象，那么必须加锁，因此性能会下降。在HotSpot虚拟机中TLABs能够解决这一问题。它允许每个线程在Eden区有自己的一小块私有空间。因为每一个线程只能访问自己的TLAB，所以在这个区域甚至可以使用无锁的指针碰撞技术进行内存分配。

我们已经对年轻代有了一个快速的浏览。你不需要要记住我刚才提到的两种技术。即便你不知道他们，也不会怎么样。但请务必记住:对象第一次被创建发生在Eden区，长期存活的对象被移动到老年代的Survivor区。

## 老年代GC
当老年代数据满时，基本上会执行一次GC。执行程序根据不同GC类型而变化，所以如果你知道不同类型的垃圾收集器，会更容易理解垃圾回收过程。

在JDK7中，有5种垃圾收集器:

Serial收集器

Parallel收集器

Parallel Old收集器 (ParallelCompacting GC)收集器

Concurrent Mark & Sweep GC  (or “CMS”)收集器

Garbage First (G1) 收集器

其中，serial 收集器一定不能用于服务器端。这个收集器类型仅应用于单核CPU桌面电脑。使用serial收集器会显着降低应用程序的性能。

现在让我们来了解每个收集器类型。

## Serial收集器
我们在前一段的解释了在年轻代发生的垃圾回收算法类型。在老年代的GC使用算法被称为“标记-清除-整理”。

该算法的第一步是在老年代标记存活的对象。

从头开始检查堆内存空间，并且只留下依然幸存的对象（清除）。

最后一步，从头开始，顺序地填满堆内存空间,将存活的对象连续存放在一起，这样堆分成两部分：一边有存放的对象，一边没有对象（整理）。

serial收集器应用于小的存储器和少量的CPU。

## Parallel收集器
![Serial收集器和 Parallel收集器的差异](http://img.blog.csdn.net/20160408192504504?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
从这幅图中，你可以很容易看到Serial收集器和 Parallel收集器的差异。serial收集器只使用一个线程来处理的GC，而parallel收集器使用多线程并行处理GC，因此更快。当有足够大的内存和大量芯数时，parallel收集器是有用的。它也被称为“吞吐量优先垃圾收集器。”

## ParallelOld 垃圾收集器
Parallel Old收集器是自JDK 5开始支持的。相比于parallel收集器，他们的唯一区别就是在老年代所执行的GC算法的不同。它执行三个步骤：标记-汇总-压缩（mark – summary – compaction）。汇总步骤与清理的不同之处在于，其将依然幸存的对象分发到GC预先处理好的不同区域，算法相对清理来说略微复杂一点。

## CMSGC
![Serial GC & CMS GC](http://img.blog.csdn.net/20160408192550192?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
## CMS垃圾收集器
如你在上图看到的那样, CMS垃圾收集器比之前我解释的各种算法都要复杂很多。初始标记（initial mark）比较简单。这一步骤只是查找距离类加载器最近的幸存对象。所以停顿时间非常短。之后的并发标记步骤，所有被幸存对象引用的对象会被确认是否已经被追踪检查。这一步的不同之处在于，在标记的过程中，其他的线程依然在执行。在重新标记步骤会修正那些在并发标记步骤中，因新增或者删除对象而导致变动的那部分标记记录。最后，在并发清除步骤，垃圾收集器执行。垃圾收集器进行垃圾收集时,其他线程的依旧在工作。一旦采取了这种GC类型，由于垃圾回收导致的停顿时间会极其短暂。CMS 收集器也被称为低延迟垃圾收集器。它经常被用在那些对于响应时间要求十分苛刻的应用上。

当然，这种GC类型在拥有stop-the-world时间很短的优点的同时，也有如下缺点：

 它会比其他GC类型占用更多的内存和CPU，默认情况下不支持压缩步骤，在使用这个GC类型之前你需要慎重考虑。如果因为内存碎片过多而导致压缩任务不得不执行，那么stop-the-world的时间要比其他任何GC类型都长，你需要考虑压缩任务的发生频率以及执行时间。
## G1GC
最后，我们来学习一下G1类型。
![Layout of G1 GC](http://img.blog.csdn.net/20160408192616239?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
如果你想要理解G1收集器，首先你要忘记你所理解的新生代和老年代。正如你在上图所看到的，每个对象被分配到不同的网格中，随后执行垃圾回收。当一个区域填满之后，对象被转移到另一个区域，并再执行一次垃圾回收。在这种垃圾回收算法中，不再有从新生代移动到老年代的三部曲。这个类型的垃圾收集算法是为了替代CMS 收集器而被创建的，因为CMS 收集器在长时间持续运行时会产生很多问题。

G1最大的好处是他的性能，他比我们在上面讨论过的任何一种GC都要快。但是在JDK 6中，他还只是一个早期试用版本。在JDK7之后才由官方正式发布。就我个人看来，NHN在将JDK 7正式投入商用之前需要很长的一段测试期（至少一年）。因此你可能需要再等一段时间。并且，我也听过几次使用了JDK 6中的G1而导致Java虚拟机宕机的事件。请耐心的等待它更稳定吧。

# 建议
通过对垃圾收集器的介绍和梳理，在管理垃圾回收方面提出了五个建议，降低收集器开销，进一步提升项目性能。

保持GC低开销最实用的建议是什么？

早有消息声称Java 9即将发布，但如今却一再推迟，其中比较值得关注的是G1（“Garbage-First”）垃圾收集器将成为HotSpot JVM的默认收集器。从串行收集器到CMS收集器，在整个生命周期中JVM已历经多代GC的实现和更新，而接下来，G1收集器将谱写新的篇章。

随着垃圾收集器的持续发展，每一代都会进行改善和提高。在串行收集器之后的并行收集器利用多核机器强大的计算能力，实现了垃圾收集多线程。而之后的CMS（Concurrent Mark-Sweep）收集器，将收集分为多个阶段执行，允许在应用线程运行同时进行大量的收集，大大降低了“stop-the-world”全局停顿的出现频率。而现在，G1在JVM上加入了大量堆和可预测的均匀停顿，有效地提升了性能。

尽管GC不断在完善，其致命弱点还是一样：多余的和不可预知的对象分配。但本文中提出了一些高效的长期实用的建议，不管你选择哪种垃圾收集器，都可以帮助你降低GC开销。

## 建议1：预测收集能力
所有的Java标准集合和大多数自定义的扩展实现（如Trove 和谷歌的Guava），都会使用底层数组（无论基于原始或基于对象）。数据的长度一旦分配后，数组就不可变了，所以在许多情况下，为集合增加项目可能会导致老的底层数组被删除，然后需要重新分配一个更大的数组来替代。

大多数的集合实现都尝试在集合没有被设置为预期大小时，还能对重分配过程进行优化，并降低其开销。但是，最好的结果还是在构造集合时就设置成预期大小。

让我们看一下下面这个简单的例子：

``` java
public static List reverse(List<?extends T> list) {
 
   List result = new ArrayList();

   for (int i = list.size() - 1; i >= 0; i--) {
       result.add(list.get(i));
    }
   return result;
}
```
以上方法分配了一个新的数组，再将另一个列表的项目填充其中，但只能按倒序填充。

但是，难就难在如何优化增加项目到新列表这一步骤。每次添加后，该列表还需确保其底层数组有足够的空槽能装下新项目。如果能装下，它就会直接在下一个空槽中存储新项目；但如果空间不够，它就会重新分配一个底层数组，将旧数组的内容复制到新数组中，然后再添加新项目。这一过程会导致分配的多个数组都会占据内存，直到GC最后来回收。

所以，我们可以在构建时告知数组需容纳多少个项目，重构后的代码如下：
``` java 
public static List reverse(List<?extends T> list) {  
   
   List result = new ArrayList(list.size());  
   
   for (int i = list.size() - 1; i >= 0; i--) {  
       result.add(list.get(i));  
    }  
   return result;  
}
```
这样一来，可以保证ArrayList构造函数在最初配置时就能容纳下list.size()个项目，这意味着它不需要再在迭代中重新分配内存。

Guava的集合类则更加先进，允许我们用一个确切数量或估计值来初始化集合。

List result =Lists.newArrayListWithCapacity(list.size());

List result =Lists.newArrayListWithExpectedSize(list.size());

第一行代码是我们知道有多少项目需要存储的情况，第二行会分配一些多余填充以适应预估误差。
## 建议2：直接用处理流
当处理数据流时，如从文件中读取数据或从网上下载数据，例如，我们通常可以从数据流中有所发现：

byte[] fileData = readFileToByteArray(newFile("myfile.txt"));

由此产生的字节数组可以被解析为XML文档、JSON对象或协议缓冲消息，来命名一些常用选项。

当处理大型或未知大小的文件时，这个想法则不适用了，因为当JVM无法分配文件大小的缓冲区时，则会出现OutOfMemoryErrors错误。

但是，即使数据大小看似能管理，当涉及到垃圾回收时，上述模式仍会造成大量开销，因为它在堆上分配了相当大的blob来容纳文件数据。

更好的处理方式是使用合适的InputStream（本例中是FileInputStream），并直接将其送到分析器，而不是提前将整个文件读到字节数组中。所有主要库会将API直接暴露给解析流，例如：
``` java 
FileInputStream fis = newFileInputStream(fileName);  
MyProtoBufMessage msg = MyProtoBufMessage.parseFrom(fis);
```

## 建议3：使用不可变对象
不变性有诸多优势，但有一个优势却极少被重视，那就是不变性对垃圾回收的影响。

不可变对象是指对象一旦创建后，其字段（本例中指非原始字段）将无法被修改。例如：
``` java
public class ObjectPair {  
   
   private final Object first;  
   private final Object second;  
   
   public ObjectPair(Object first, Object second) {  
       this.first = first;  
       this.second = second;  
    }  
   
    publicObject getFirst() {  
       return first;  
    }  
   
   public Object getSecond() {  
       return second;  
    }  
   
}
```
实例化上面类的结果为不可变对象——所有的字段一旦标记后则不能再被修改。

不变性意味着在构造容器完成之前，由不可变容器引用的所有对象都已经创建。在GC看来：容器会和其最新的新生代保持一致。这意味着当对新生代（young generations）执行垃圾回收周期时，GC可以跳过老年代（older generations）中的不可变对象，因为它知道不可变对象不能引用新生代的任何内容。

越少对象扫描意味着需扫描的内存页越少，而越少的内存页扫描意味着GC周期越短，同时也预示着更短的GC停顿和更好的整体吞吐量。

## 建议4：慎用字符串连接
字符串可能是任何基于JVM的应用中最普遍的非原始数据结构。但是，其隐含重量和使用便利性使得它们成为应用内存变大的罪魁祸首。

很明显，问题不在于被内联和拘留的文字字符串，而在于字符串在运行时被分配和构建。接下来看看构建动态字符串的简单示例：
``` java
public static String toString(T[] array) {  
   
   String result = "[";  
   
   for (int i = 0; i < array.length; i++) {  
       result += (array[i] == array ? "this" : array[i]);  
       if (i < array.length - 1) {  
           result += ", ";  
       }  
    }  
   
   result += "]";  
   
   return result;  
}
```
获取数组并返回它的字符串表示是一个很不错的方法，但这也正是对象分配的问题所在。

要看到其背后所有的语法糖并不容易，但真正的幕后场景应该是这样：
``` java
public static String toString(T[] array) {  
   
   String result = "[";  
   
   for (int i = 0; i < array.length; i++) {  
   
       StringBuilder sb1 = new StringBuilder(result);  
       sb1.append(array[i] == array ? "this" : array[i]);  
       result = sb1.toString();  
   
       if (i < array.length - 1) {  
           StringBuilder sb2 = new StringBuilder(result);  
           sb2.append(", ");  
           result = sb2.toString();  
       }  
    }  
   
   StringBuilder sb3 = new StringBuilder(result);  
   sb3.append("]");  
   result = sb3.toString();  
   
   return result;  
}
```

字符串是不可变的，所以在其连接时并没有被修改，而是依次分配新的字符串。此外，编译器利用标准StringBuilder类来执行的这些链接。这就导致了双重麻烦，在每次循环迭代时，我们得到（1）隐式分配临时字符串，（2）隐式分配临时的StringBuilder对象来帮助我们构建最终结果。

避免上述问题的最佳方法是明确使用StringBuilder并直接附加给它，而不是使用略幼稚的串联运算符（“+”）。所以应该是这样：

``` java 
public static String toString(T[] array) {  
   
   StringBuilder sb = new StringBuilder("[");  
   
   for (int i = 0; i < array.length; i++) {  
       sb.append(array[i] == array ? "this" : array[i]);  
       if (i < array.length - 1) {  
           sb.append(", ");  
       }  
    }  
   
   sb.append("]");  
   return sb.toString();  
}
```

此时，在方法开始时我们只分配了StringBuilder。从这一点来看，所有的字符串和列表项都会被添加到唯一的StringBuilder中，最终只调用一次toString方法转换成字符串，然后返回结果。

## 建议5：使用专门的原始集合
Java的标准库非常方便且通用，支持使用集合绑定半静态类型。例如，如果要用一组字符串（Set<String>），或一对字符串映射到字符串列表（Map<Pair, List<String>>），直接利用标准库会非常方便。

事实上，问题之所以出现是因为我们想把double类型的值放在 int 类型的list集合或map映射中。由于泛型不能调用原始集合，则可以用包装类型代替，所以放弃List<int>而使用List<Integer>更好。

但其实这非常浪费，Integer本身就是一个完备对象，由12字节的对象头和内部4字节的整数字段组合而成，加起来每个Integer对象占16个字节，这是同样大小的基类int类型长度的4倍！然而，更大的问题是所有这些Integer实际上都是垃圾回收过程中的对象实例。

为了解决这个问题，我们在Takipi 中使用优秀Trove 集合库。Trove放弃了一些（但不是全部）支持专业高效内存的原始集合的泛型。例如，不用浪费的Map
``` java
TIntDoubleMap map = newTIntDoubleHashMap();  
map.put(5, 7.0);  
map.put(-1, 9.999);  
...
```

Trove底层实现了原始数组的使用，所以在操作集合时没有装箱（int -> Integer）或拆箱（Integer -> int）发生，因此也不会将对象存储在基类中。


# 业务思想
随着垃圾收集器不断进步，以及实时优化和JIT编译器变得更加智能，作为开发者的我们，可以越来越少地操心代码的GC友好性。尽管如此，无论G1有多先进，在提高JVM方面，我们还有许多问题需要不断探索和实践，百尺竿头仍需更进一步。

 

参考资料：

《深入理解Java虚拟机》第2版·周志明

伯乐在线·网站学习整理

OneAPM官方技术学习整理
