---
title: 【转载】PhantomReference and Finalization
date: 2016-04-05 21:27:34
modified: 2016-04-05 21:27:34
author: jojoster
postid: 145
slug: 145
linktitle: phantomreference and finalization
attachments: $ATTACHMENTS
posttype: post
poststatus: publish
categories: translate
tags: ['译文']
category: java
---

本文翻译至：[PhantomReference and Finalization][1]
<!--more-->

## 软引用，弱引用以及虚引用

软引用（`SoftReferences`），比较典型的应用是在内存缓存的场景中。JVM会尽可能地将对象保留在内存中，当JVM内存不足的时候，才会从最早的references开始清除。根据javadoc中的描述，整个清除过程是没有保障的。

弱引用（`WeakReferences`）是我最经常使用的类型。典型的用途是在创建一些弱引用的监听器（Listener），或者是想收集某个对象的额外信息（使用WeakHashMap）的场景中。非常有助于降低[类耦合度][2]


 
其实笔者读到这里的时候，是产生了一些疑问的。为何使用weakHashMap可以降低类耦合度？设想一下使用了WeakHashMap的场景，weakhashmap可以优雅的解决内存释放的问题，但是如果没有WeakHashMap的话，那么实现就会复杂许多。可以在对象不在使用的时候，将它从Map中移除。这就需要容易管理者构造一个清理的函数给对象调用者使用，或者使用一个监听器模式。比如在编写一些使用者非常广泛的api类型的代码时候（比如jdk的api），添加这样的函数可能会使使用者的api变得非常复杂。

虚引用（`Phantom references`）则适用于在垃圾回收之前进行的预处理，比如需要释放一些资源的场景。遗憾的是，很多开发者会使用`finalize()`方法去执行这些操作，这并不是一个好的方式。finalize方法如果没有小心的使用在恰当的线程，恰当的时机，那么很可能会对应用造成可怕的性能影响，甚至会影响应用的数据完整性。

在虚引用的构造方法中，开发者需要显式的指定一个`ReferenceQueue`去将已经标记为“phantom reachable”的对象加入ReferenceQueue队列中。“phantom reachable”代表连虚引用本身都引用不到的对象。最令人迷惑的是即使`Phantom references`继续保持着私有对象的引用（区别于软引用以及弱引用），`get`方法也会返回一个null。这样一来，一旦进入这个状态的对象就无法再一次获得强引用。

开发者可以一次一次的对`ReferenceQueue`调用`poll()`方法，检测是否有新的`PhantomReferences`进入“phantom reachable”状态。正常的写法中，可以使类继承于`java.lang.ref.PhantomReference`，以保证引用的对象只垃圾回收一次，然后无法继续被获取。

## `PhantomReference` 以及 `finalization`的细节

对PhantomReference 来说，最常见的误解会认为它是被设计用来“修复”finalizers 带来的对象逃逸问题。举个例子来说，我们常常会这么说：

虚引用可以避免`finalize()`带来的基础问题：**`finalize()`方法可以通过创建一个新的强引用，使自身免于垃圾回收而进行“逃逸”**。所以，重写了`finalize()`方法的对象，需要至少在两条分别的垃圾回收链中，才会被正确的回收。

然而，使用了虚引用，也有可能使对象出现逃逸，请看以下的代码
```java
Reference ref = referenceQueue.remove();  //ref is our PhantomReference instance
Field f = Reference.class.getDeclaredField("referent");
f.setAccessible(true);
System.out.println("I see dead objects! --> " + f.get(ref)); //This is obviously a very bad practice.
```
由此可见，表面上看，引用类型非常有可能是通过成员变量 `Reference#referent` 指向那些已经失去引用的对象。但是实际上，垃圾回收器对对特定的对象产生了一个意外。这一现象也直接对上文中的结论产生了冲突：
> 虚引用只用对象在实际的内存空间中被移除时候，才会执行`enqueued`操作。

到底哪种说法是正确的， javadoc是这样说明的：

> Phantom references are most often used for scheduling pre-mortem cleanup

所以，如果虚引用并不是设计用来修正finalize逃逸的问题（这个问题非常严肃，曾经被Lazarus、Jesus 以及许多其他学者指出），那么虚引用究竟有什么作用？

`finalize()`方法实际是通过垃圾回收线程去执行的，即使在简单的单线程应用中，考虑到潜在因素，也可能出现并发问题（比如错误的将共享状态放入同步方法中等）。但是使用了虚引用的话，你可以制定执行出队操作的线程（在单线程程序中，指定的线程会周期性的做这个任务）

## 使用 WeakReference 的话，会如何？

弱引用看起来也会满足垃圾回收之前的内存清理场景。区别在于合适进行引用的入队操作。PhantomReference会在执行`finalization`之后入队，而WeakReference会在之前。对于`finalize()`方法中没有关键实现的对象来说，不受影响。

但是对于那些需要在`finalize()`方法中执行一些清理的对象，就会有些许不同

(PhantomReference's get() 方法总是返回null)。开发者需要存储尽可能多的状态信息，去进行清理操作。举个例子，清理array中的对象，设置为null以后，开发者需要记录下来array中对象的下标，方便后续跟踪查看。对于这类型操作，可以将类继承于PhantomReference，然后创建这个类的实例。

下面更进一步的说说。

想象一个场景：一名开发者准备在某个对象中编写一段清除钩子的代码（通过 finalize()或者是通过[Weak|Phantom]Reference），当这个对象仅仅有属于线程栈空间的强引用（比如局部变量）的时候，开发者调用了一个方法，那么这时，可能发生这样的事情：

出于性能的考虑，JVM会检测是否这个对象有失去引用的可能。所以，在执行方法的过程中，finalization 可能被并发的执行。这样可能导致一些不可预料的结果（finalization 可能修改了一些类内部的状态，比如其他方法也会使用这些状态）。这种情况非常罕见，可以采取以下的方式修复：

```java
Object method() { 
	synchronized (this) {//do work here }
	return result;
}

public void finalize() {
	synchronized (this) { //do work here}
}
```
这种情况仅仅适用于那些仅仅在线程栈中持有的对象：
- 对象重写了`finalize()`方法。
- 有一个[Weak|Soft|PhantomReference]引用指向这个对象，同时已经进入了`ReferenceQueue`，有另外一个线程进行dequeue的操作

总结一下，最安全的清理机制，是通过`PhantomReference`以及`ReferenceQueue`，在同一个线程下进行清理。如果是启用了另一个线程，那么就需要使用同步方法快


[1]: http://www.erpgreat.com/java/phantomreference-and-finalization.htm "原文"
[2]:  http://www.xiaoyaochong.net/wordpress/index.php/2013/08/05/java%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2%E4%B8%8Eweakhashmap// "如何利用hashMap降低类耦合度"