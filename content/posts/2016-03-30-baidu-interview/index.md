---
title: 百度面试心得
date: 2016-03-30 22:58:10
modified: 2016-03-30 22:58:10
author: jojoster
postid: 108
slug: 108
nicename: interview-baidu
attachments: 140
posttype: post
poststatus: draft
tags: ['tech','life','面试']
category: interview
---


经过朋友内推，终于获得了百度的面试机会， 这里主要介绍技术面试的内容
技术面试主要分为两轮面试，整体感觉百度面试的比较详细，很多细节方面的东西

技术面试通过以后，会有另外一个面试
百度的面试效率比较高，三轮面试总共持续了2天

<!--more-->

###第一轮面试


> #### 1. 简单介绍自己

这个没什么说的，主要都是从简历上面说一下，面试官也会根据简历进行提问

>#### 2. 想象一个场景：一个系统要同步美国（订单）和中国的数据（其余主要数据），通过太平洋海底光缆进行传输，成本非常高。现在遇到了一些问题，随着数据量增加，无法在规定时间内进行同步，现在设计一种同步方式，达到目的（提示:MQ）。

1. **开多个端口，并行传输**
2. 通过专门的消息中间件(MQ)进行生产者消费者模式的数据推送和获取
3. MQ应该部署在美国，保证下单服务的平滑和正常
4. **一致性方面，考虑使用MQ的回执（[Acknowledgment][1]）来实现**

> ####3. threadlocal以及其应用场景

- threadLocal是多线程的概念，主要指的是存储于线程中的对象，属于线程特有，同一个线程执行的所有方法， 都可以获取这部分数据。
- threadLocal中的数据只能有本身的线程访问，所以一般情况下， 这部分数据是线程安全的
- hibernate中的[thread-bind-session][2]就使用了threadLocal，当然还有其他实现
- spring的 [OpenSessionInViewFilter][13] 机制

>#### 4. 利用tomcat做过什么实现?jetty中的nio是怎么体现的?

jetty中，异步的配置主要体现在这里：
```xml
 <Call name="addConnector">
      <Arg>
          <New class="org.mortbay.jetty.nio.SelectChannelConnector">
            <Set name="port"><SystemProperty name="jetty.port" default="8080"/></Set>
            <Set name="maxIdleTime">30000</Set>
            <Set name="Acceptors">2</Set>
            <Set name="confidentialPort">8443</Set>
          </New>
      </Arg>
    </Call>
```
nio在异步请求模型中的作用主要是将模型的三大部分都通过异步的方式处理
1. 侦听链接线程
2. 侦听请求线程
3. 数据处理线程

参见博客原文: <[从Jetty、Tomcat和Mina中提炼NIO构架网络服务器的经典模型][3]>

主要应用到的nio包结构如下:
参见博客：<[仿照jetty的nio原理写了个例子][4]>
```java
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector; // 起到阻塞的作用, selector.select()
import java.nio.channels.ServerSocketChannel; // 通过channel绑定到一个端口，
import java.nio.channels.SocketChannel; //
import java.util.concurrent.ConcurrentLinkedQueue; //两个实例, 主要存放接收到的 链接 和 请求
```
主要监听方法, 每个步骤都是一个线程去处理的, 主线程不会阻塞
```java
public void listen() throws IOException { // 服务器开始监听端口，提供服务
        channel.socket().bind(new InetSocketAddress(port)); // 将scoket榜定在制定的端口上
        channel.configureBlocking(true);
        new Thread(new ConnectionHander()).start();
        new Thread(new RequestExecutor()).start();
        new Thread(new RequestHander()).start();
}
```

对于nio相关的类的解释, 参见这个博客:<[Java NIO API详解][5]>

>#### 5. 算法：一个数组中有多个整数， 其中有两个重复的数字，如何找出来，考虑一下时间复杂度和空间复杂度

这个算法的解释比较多
1. 位图法/Hash, 空间复杂度为n, 时间复杂度为n
2. 先对数组排序, 然后再进行遍历, 时间复杂度>n

>#### 6. 系统优化总共分为几个部分？如何优化？

>#### 7. Hash函数的概念

>#### 8. hash函数以及简单的应用
>#### 9. OSGI模型, 以及其中的应用

osgi指的是 **Open Services Gatewag initiative**，是一套java提供的一种更加抽象和封装性的接口，有一些特性非常重要：
1. 动态性，可以动态的升级模块
2. 基于接口编程，完全隐藏实现
3. 提供了更加严格的模块检查，更加严格的一致性检查

具体参见这个[教程][14]，需要登录


###第二轮面试
######(不按照时间顺序排列)
>#### 1. 白板编写程序: 生产者消费者模型 或者 单例模型（主要考查单例的写法， 两个判空）

作者选取了单利模型的编写，单例以及其写法讨论， 参见这个博客：[《Java Notes 00 - Singleton Pattern(单例总结)》][6]
本来作者给出了内部类的答案，但是面试官还是偏向想看线程安全的写法

> #### 2. spring的事务是怎么实现的，嵌套场景下的aop实现

通过aop做代理实现，但是需要强调的是：
1. 一般场景下， 需要通过spring注入的方法才会代理，同一个类内部调用方法的话，是不会产生代理作用的
  -  *但是事务回滚的话，是通过异常去处理的？内部方法抛出异常的话，会回滚吗*
2. 对于需要全部进行代理的，需要了解一下 [LTW][7] 的概念 ,或者了解一下[Javassist][8]的包
  - 推荐一个git:[simpleApm][9]

嵌套事务的话, 了解一下配置,会有一些概念 参见博客: <[关于Spring嵌套事务][10]>

- PROPAGATION_REQUIRED -- 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
- PROPAGATION_SUPPORTS -- 支持当前事务，如果当前没有事务，就以非事务方式执行。
- PROPAGATION_MANDATORY -- 支持当前事务，如果当前没有事务，就抛出异常。
- PROPAGATION_REQUIRES_NEW -- 新建事务，如果当前存在事务，把当前事务挂起。
- PROPAGATION_NOT_SUPPORTED -- 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER -- 以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED -- 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。
- 前六个策略类似于EJB CMT，第七个（PROPAGATION_NESTED）是Spring所提供的一个特殊变量。
它要求事务管理器或者使用JDBC 3.0 Savepoint API提供嵌套事务行为（如Spring的DataSourceTransactionManager）
```java
public void listen() throws IOException { // 服务器开始监听端口，提供服务
        channel.socket().bind(new InetSocketAddress(port)); // 将scoket榜定在制定的端口上
        channel.configureBlocking(true);
        new Thread(new ConnectionHander()).start();
        new Thre![](http://)ad(new RequestExecutor()).start();
        new Thread(new RequestHander()).start();
}
```

> #### 3. 描述一个项目及其架构
> #### 4. ES的使用, 使用中有没有做过数据量的测试
> #### 5. 数据库调优方面，有什么方法

1. 主要先查看瓶颈在哪里，比如CPU，内存，磁盘
2. 根据业务的数据量，确定数据库设计是否合理
3. 使用工具捕获执行时间长的语句，然后调优，根据业务场景查看语句写法是否恰当

> #### 6. JVM中，Minor GC、Major GC和Full GC之间的区别

推荐一个博客:<[hunter129][11]>

> #### 7. JVM中，你遇到的内存溢出是什么情况的，怎么发现的，怎么调查的，怎么修改的，如何保证后续的稳定

java中, 针对jvm的状态监测， 在windows的调试中， 最简单的也许是可视化工具 jvisualvm.exe
一般来说，通过jvisuavm可以比较方便的观察jvm的所有情况，包括cpu、线程、内存等

> #### 8. threadlocal的概念，线程安全的概念，synchronized的理解（这个关键字是锁方法还是锁对象）

**坑爹的是, synchronized这个关键字整个是锁对象的!  参见一篇文章 [java同步设计败笔][12]**
顺便推荐一个公众号 微信号 iteedu 很少人知道, 但是很好

> #### 9.  线程的概念，使用多线程的场景是什么样的？线程池是怎么实现的？(BlockQueue的概念)

线程是程序的实际工作者，不同于进程，进程更加偏向于资源的管理。跨线程之间的数据交换比较简单，但是跨进程就会比较艰难。
使用多线程的场景往往是有等待的情况。等待可以来源于很多情况，比如cpu等待io，UI线程等待后台线程等等。
多线程的正确使用， 往往会带来硬件的充分利用。
但是多线程的使用， 往往也会带来一些线程安全的问题。
此时就需要充分利用锁的概念。

> #### 10. 遇到了一个服务，发现有性能问题，如何在现有的机器之下做最大化的优化？从哪些方面着手？
> #### 11. redis中的所有数据结构，redis是怎么用的？集群策略的话，需要考虑哪些东西，redis代理用过不？

{{ $image := .Resources.Get "redis-cluster-art.jpg" }}

######redis集群策略
- 官方自带 : redis-cluster 需要redis-3.0以上的版本
  - 参见redis的官方[文档][18]，这个集群的实现，通过在redis中增加集群配置，然后每个节点都可以向任意节点发送请求
  - 每个get和set操作都会事先进行hash映射，然后到目的节点中执行
  - ![redis-cluster结构图]({{$image.RelPermalink}})
- 开源的集群代理主要的实现有[dynomite][15]，[twemproxy][16]，[codis][17]
  - 代理则不一致，代理主要面对的是另一种应用场所
  - ![twemproxy的部署图]({{ $image := .Resources.Get "twemproxy-deploy.jpg" }})

###### redis

> #### 12. springMVC的设计模式，都是做什么的？ Controller是使用Servlet规范中哪个对象实现的？

Controller是使用filter实现的
> #### 13. spring事务的回滚机制，是怎么回滚的？子方法抛出异常的话，会回滚吗？

事务回滚， 必须要抛出异常，异常要是被捕获的话，事务是不会回滚的，这个要基于spring的事务实现机制进行考虑。

> #### 14. redis的数据类型，list和set的区别？

redis中5种数据类型
1. string， 主要的操作包括 get， set，incr，decr，mget以外， 还有一些其他操作
  - 获取字符串长度
  - append
  - 设置和获取字符串的某一内容
  - 设置和获取某一bit
  - 批量设置一系列字符串的内容
2. hash
3. list 是一个双向列表，主要有以下操作
  - l操作，队列的头进行操作
  - r操作，队列的尾操作
  - bl操作， 阻塞式l操作
4. set，本质上是一个hashmap的实现
5. sort set 有加权以后的hashmap实现，根据权重 score来进行排序

> #### 15. 说说NIO的概念，与普通IO有什么区别？
> #### 16. 数据复制，数据迁移方面是怎么做的？用什么工具？mysql的binlog是怎么应用的？有什么开源框架在使用这个特性？

binlog：参见这个[介绍][19]

###第三轮面试

第三轮面试主要偏向对人的考查，比如抗压能力，对加班的看法，对百度的看法等。估计面试官不一样，考查方式也不一样。
面试官会比较偏重于第一映像，同时会察言观色

> #### 1. 编写一个二分搜索的程序，一张纸，一支笔直接写

这个程序还是十分简单的，面试你官不仅会查看编写程序的结果，也会看着你编程，其中会测试你的抗压能力

>#### 2. 对996加班机制怎么看？对年纪比较小，但是收入要更高的其他成员会怎么看？
>#### 3. 针对百度目前的舆论是怎么看的

[1]: https://www.ibm.com/support/knowledgecenter/SSFKSJ_8.0.0/com.ibm.mq.msc.doc/xms_cmesack.htm        "IBM的解释"
[2]:  http://grepcode.com/file/repo1.maven.org/maven2/org.hibernate/hibernate/3.2.7.ga/org/hibernate/context/ThreadLocalSessionContext.java?av=h#ThreadLocalSessionContext  "grepcode源码"
[3]: http://blog.csdn.net/cutesource/article/details/6192016 "博客: 走向架构师之路"
[4]: http://daizuan.iteye.com/blog/1112909 "博客: 记录每一次收获"
[5]: http://www.blogjava.net/19851985lili/articles/93524.html "博客: 永恒的瞬间"
[6]: http://hukai.me/java-notes-singleton-pattern/ "博客: 胡凯"
[7]: http://www.eclipse.org/aspectj/doc/next/devguide/ltw.html "aspectj的 load-time weaving"
[8]: http://jboss-javassist.github.io/javassist/ "java的动态编程"
[9]: https://github.com/dingjs/simpleApm/
[10]: http://zhongl.iteye.com/blog/293161 "唯有思考没停"
[11]: http://static.xiegq.com/
[12]: http://mp.weixin.qq.com/s?__biz=MzIzNTEwNDM5Mg==&mid=401120022&idx=1&sn=76566b3eb443668f0a51f2a3144aff86&scene=1&srcid=033168usDJS51iJoxCjHVBlH#wechat_redirect
[13]: http://www.iteye.com/topic/32001
[14]: http://course.tianmaying.com/osgi-toturial/lesson/osgi-concept
[15]: https://github.com/Netflix/dynomite "netflix的开源实现，针对亚马逊的Dynamo"
[16]: https://github.com/twitter/twemproxy "twitter的实现"
[17]: https://github.com/CodisLabs/codis "豌豆荚的实现"
[18]: http://www.redis.cn/topics/cluster-tutorial.html
[19]: http://www.cnblogs.com/Richardzhu/p/3225254.html
