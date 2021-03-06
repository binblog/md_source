---
title: java 引用
tags: java
---

Java中有如下四种类型的引用：  

+ 强引用(Strong Reference)  
+ 软引用(SoftReference)  
+ 弱引用(WeakReference)  
+ 虚引用(PhantomReference)

强引用就是指在程序代码之中普遍存在的，类似`Object obj = new Object()`之类的引用，只要强引用还存在，垃圾收集器永远不会回收被引用的对象。

虚引用也称为幽灵引用或绝影引用，它是最弱的一种引用关系。一个对象是否有虚引用，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。  

虚引用主要用来跟踪对象被垃圾回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。    	


MaxMetaspaceSize