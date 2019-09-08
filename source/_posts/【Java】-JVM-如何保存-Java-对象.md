title: 【Java】 JVM 如何保存 Java 对象
date: 2019-09-08 14:59:01
tags: JVM
categories: Java
---

## 前言
本文主要讲解一下在 JVM 中如何保存 Java 对象以及 Java 对象指针压缩相关的东西。

## JVM 体系结构
![](/images/jvm-java/jvm体系.gif)
> 图片摘自 https://www.artima.com/insidejvm/ed2/jvm2.html

<!-- more -->

上图是 JVM 规范中定义的体系结构（这个只是定义的规范，实际的  JVM 实现中可能与这个结构会有差异），这里我们主要看下运行时数据区（runtime data areas）的内容，以下摘自 [The Java® Virtual Machine Specification Java SE 8 Edition](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)
* PC register -  If that method is not native , the pc register contains the address of the Java Virtual Machine instruction currently being executed.
* Java stacks - A Java Virtual Machine stack is analogous to the stack of a conventional language such as C: it holds local variables and partial results, and plays a part in method invocation and return.
* Heap - The heap is the run-time data area from which memory for all class instances and arrays is allocated.
* Method area -  The method area is analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an operating system process. It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors, including the special methods used in class and instance initialization and interface
initialization.
* Native method stacks - An implementation of the Java Virtual Machine may use conventional stacks, colloquially called "C stacks," to support native methods (methods written in a language other than the Java programming language). Native method stacks may also be used by the implementation of an interpreter for the Java Virtual Machine's instruction set in a language such as C。

堆和方法区是所有类共享的，其中堆主要存储对象实体，方法区存储的信息比较多，主要包括下面几类：
+ 类的基本类型信息
    - 类型的全限定名
    - 直接超类的全限定名（除了 Object）
    - 是类还是接口
    - 访问修饰符
- 该类的常量池
    - 虚拟机会为每个转载的类型维护一个常量池
- 字段信息
    - 字段名称
    - 字段类型
    - 字段修饰符（public，private，protected，static，final，volatile，transient）
- 方法信息
    - 方法名
    - 方法的返回值类型或者void
    - 方法的参数数量和类型（按照声明顺序）
    - 方法的修饰符（public，private，protected，static，final，synchronized，natvie，abstract）
    - 如果不是abstract和native方法，还会保存下面的信息
      - 方法的字节码
      - 操作数栈和局部变量区的大小
      - 异常表
- 类（静态）变量
    - 静态常量和非静态常量的处理方式不同，每个类都会把用到的其他类的静态常量拷贝到自己的常量池中。
- 指向 ClassLoader 类的引用。
- 指向 Class 类的引用，对于每个被装载的类型，JVM 都会为其创建一个 java.lang.Classs 类的实例（该实例存在heap中），并且JVM 会以某种方式将该实例和方法区中对应的类型关联起来。

## 对象如何保存
我们知道一个Java对象包含两部分内容，字段和方法，每个对象的字段值都可能不同，但是所用的方法都是一样的，如果每个对象都保存一套方法定义，显然会浪费很多的空间。所以方法定义相关的都放到了方法区，对象只保存自己的实例数据和指向方法定义的指针。下图是对象保存的一种方式，也是 Hotspot 虚拟机采用的方式，对象在堆中只保存实例的数据，同时会有一个指针指向方法区中的一个方法表（和 c++ 中的 [Virtual method table](https://en.wikipedia.org/wiki/Virtual_method_table) 类似）。方法表保存两个部分：指向类数据的指针和执行各个方法的指针。这里将类数据和方法分开存储，是为了更加快速的找到方法。每个类都会对应一个方法表，这种实现方式会稍微浪费一些内存，但是会获得更好的性能。

![](/images/jvm-java/对象存储.gif)
> 图片摘自 https://www.artima.com/insidejvm/ed2/jvm6.html

我们知道对象是有继承关系的，如果子类没有覆写父类的方法，那么子类会指向父类的中的方法。

![](/images/jvm-java/method-demo.gif)
> 图片摘自 https://www.artima.com/insidejvm/ed2/linkmod12.html

## HotSpot 内存结构
上面主要是 Java 虚拟机规范中定义的规范，每种虚拟机实现的方式可能不太相同，这里我们主要看下 HotSpot 虚拟机的实现，后面的内容都是基于 HotSpot 虚拟机。
在 Java8 中，HotSpot VM 移除了永生代（PermGen），添加了元数据空间（Metaspace），元空间不使用虚拟机内存，而是使用本地内存。元空间主要和方法区对应，存储类的元数据和常量池（String常量的实例存在堆中）等信息。

![](/images/jvm-java/hotspot.jpg)
> 图片摘自 http://java-latte.blogspot.com/2014/03/metaspace-in-java-8.html

##  Ordinary Object Pointer (OOP)
在 JVM 中 Java 对象使用 OOP（Ordinary Object Pointer） 来表示，格式如下图所示。

![](/images/jvm-java/oop.png)

OOP 主要包含两个部分：对象头和实例数据。对象头主要包含四个部分：
+ Mark Word - 会存储对象的多种标记信息，例如哈希值、GC标记、锁等信息
+ Klass Word - 主要指向类的元数据
+ 32-bit length word - 只有数组对象才有，记录数组的长度
+ 32-bit gap - Java 是 8 字节对齐的（关于为什么要进行内存对齐，可以参考 [这篇文章](http://blog.zhangjikai.com/2015/11/28/%E3%80%90C%E3%80%91alignment/)），该字段主要用做对齐填充用

对象头后面就是实例数据，可能是基本数据，也可能是指向其他对象的引用。如果实例数据的大小不是 8 的倍数，那么也会插入一些填充的数据来对齐。对于继承的情况，会先存放父类的实例数据，然后再存放子类的实例数据，如下图所以：

![](/images/jvm-java/instance.png)

Mark word、Klass word 以及对象的引用大小和 JVM 位数相关，32位 JVM 是 4 字节大小，64 位 JVM 是 8 字节大小。对于引用来说，4字节来寻址的话的最多可以表示 2<sup>32</sup>，也就是做大只能支持 4GB 的内存，一般来说4GB 的内存是不大够用的，所以我们常用的是 64 位的 JVM，但是使用 64 位 JVM 带来的一个问题就是引用从 4 个字节变成了 8 个字节，也就是会多占一倍的空间，这样会导致更加频繁的 GC 周期，导致性能变差。

## Compressed OOPs
我们使用压缩的 OOP 来实现在64位的 JVM 上使用32位大小的引用来寻址，这个方式主要是基于 Java 对象是 8 字节对齐，即后三位全部为 0，也就是在当前的对象引用中后三位实际上是没有用到的。基于上面的逻辑，我们就可以做一下优化，将当前32位值的表示为第 4-35 位的值，也就是实际的值相当于左移了三位，如下图所示。这样我们就有35位来寻址，内存最大就可以支持到 32GB。

![](/images/jvm-java/compress.jpg)
> 图片摘自 https://www.baeldung.com/jvm-compressed-oops

开启了压缩之后，堆中 OOP 里的下列字段会被压缩：
+ 每个对象的 Kclass 字段（Mark不会压缩）
+ 指向其他 OOP 的引用
+ OOP 数组中的每个元素

下面是 Integer 对象在不同情况下占的内存大小，因为 Java 是 8 字节对齐，所以在64位 VM 上未开启压缩时，Integer 还要加上一个 32bit 填充，即总的大小是 192 bit。

![](/images/jvm-java/compress-demo.jpg)
> 图片摘自 https://www.javacodegeeks.com/2016/05/compressedoops-introduction-compressed-references-java.html

我们可以在启动 Java 程序时使用 `-XX:+UseCompressedOops` 来开启压缩，Java7之后，如果最大内存小于32G，会自动开启 OOP 压缩。如果想在超过 32G 内存的情况下使用压缩，可以通过指定Java 对象对齐的字节数来实现 `-XX:ObjectAlignmentInBytes`，该值必须在 8 到 256 之间，并且是 2 的指数倍。假设指定为 16，那么就可以使用 64G 的内存，但是由于对齐造成的内存浪费也会更多。

另外在 Java11 中添加的 ZGC 垃圾回收器必须使用 64 位的指针，所以它不支持压缩的OOP。

## 参考文章
+ [Inside the Java Virtual Machine](https://www.artima.com/insidejvm/ed2/)
+ [Compressed OOPs in the JVM](https://www.baeldung.com/jvm-compressed-oops)
+ [Getting Started with HotSpot and OpenJDK](https://www.infoq.com/articles/Introduction-to-HotSpot/)
+ [Know Thy Java Object Memory Layout](http://psy-lob-saw.blogspot.com/2013/05/know-thy-java-object-memory-layout.html)
+ [CompressedOops](https://wiki.openjdk.java.net/display/HotSpot/CompressedOops)
+ [CompressedOops: Introduction to compressed references in Java](https://www.javacodegeeks.com/2016/05/compressedoops-introduction-compressed-references-java.html)
+ [Metaspace in Java 8](http://java-latte.blogspot.com/2014/03/metaspace-in-java-8.html)
