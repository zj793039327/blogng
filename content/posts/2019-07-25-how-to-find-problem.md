---
author: j56
layout: post
title:  "记一次数据库问题定位"
date: 2019-07-25 17:09:22
categories: tech
---
记录一次排查数据库insert的问题
<!--more-->
## 引入

最近遇到一个服务问题，简单的描述如下：

1. 00:00:01服务A成功向数据库中insert了一条记录；
2. 00:00:02 服务B执行数据库查询，查询为空。

对于这个问题，我怀疑问题产生可能因为下列原因：

1. 服务本身有问题，服务A insert 动作在 服务B查询之后：
   1. 可能是内部bug
   2. 可能是事务原因，比如A在inert中开启了事务，00:00:02之后才执行commit。
2. 数据库主从延迟，比如服务A写入了主库，服务B却读取了从库，如果主从延迟到1s，那么上述状况属于正常。

接下来就根据这俩思路进行调查，最终还是怀疑db延迟，但是也说下调查中发现的一些有意思的点。







## 详细问题

首先扫一遍代码，确定代码的执行顺序，明确B的查询肯定是在A的insert之后。那么只能是事务问题了。

在调查事务的前提是如何知道代码在执行中开启了事务呢，一个最简单的方案看这里-北京瓜的博客-《[启用mysql日志记录执行过的sql》](https://www.cnblogs.com/bjgua/p/8376363.html)

简单来说就是在mysql中开启sql日志，通过`SET autocommit=0` 来判断是否开启事务。

我这里复述下开启sql日志的过程，本质上是在mysql中开启日志配置。

临时开启日志的话，下列两个命令看下

```sql
SHOW VARIABLES LIKE "general_log%";
```

![](http://img.skydrift.cn/1564047471.png)

如果general_log=OFF的话，执行下面的sql开启即可。

```sql
SET GLOBAL general_log_file = '/var/lib/mysql/localhost.log';
SET GLOBAL general_log = 'ON';
```

开启之后，可以看到所有的sql都会打印到日志中。

![](http://img.skydrift.cn/1564047589.png)

事务和非事务的区别，就在于是否 autocommit=1。

回到服务中，查看服务的代码，看到服务代码中使用了`@Transactional`注解，但是实际的执行当中，没有开启事务。

通过sql日志可以看到，最终还是` SET autocommit=1`

![](http://img.skydrift.cn/1564053255.png)

但是有意思的地方在于，这一次执行当中，有3次 `SET autocommit`。

回看代码，发现有一个advice很有意思。

```xml
    <aop:config>
        <!-- 分表默认不加所有的事务-->
        <aop:pointcut id="tableMapperOperation"
                      expression="execution(* com.x.*XMapper.*(..))"/>
        <aop:advisor advice-ref="noTxAdvice" pointcut-ref="tableMapperOperation"/>
    </aop:config>
    <!-- 此处的 transaction-manager 可以不指定，默认找 transactionManager-->
    <tx:advice id="noTxAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="NOT_SUPPORTED"/>
        </tx:attributes>
    </tx:advice>
```

实际的方法栈是长这样子的：

![](http://img.skydrift.cn/1564055328.png)

可以看到，一个简单的controller-service-dao的调用链，使用了spring框架，增加了若干拦截器之后，变成了如此复杂的结构。

注意其中画红框的部分：

1. controller层自不必说，肯定是一个代理，因为要统计日志，统一处理异常等。
2. service层注意，由于有doInTransaction，这里service本体是个代理，而且还是`xxx$$EnhancerBySpringCGLIB`，说明是使用CGLIB生成的代理。上图是使用`编程型事务（TransactionTemplate）`的调用方法栈截图，下图则是使用了` @Transactional 注解` 引入事务的截图。![@Transactional注解事务方法栈](http://img.skydrift.cn/1564067104.png)
3. dao层，这就是上面那个noTxAdvice发挥作用的时候了，可以看到，这里的mapper其实是个代理，是使用jdk生成的代理`jdkDynamicAopProxy`。

答案比较清晰了，首先使用 `@Transactional`在service层配置了事务，但是在dao层，又通过xml配置型advice设置了不开事务，最终dao层的代码先执行，没开事务。

要问为啥代码这个样子，挖坑不填，出来混，总是要还的。

## more than @Transactional

既然发现了这么有意思的东西，突然想看下 事务中 不同的 Propagation（事务传播）在数据库层面会如何表示和实现。

### Propagation取值

#### REQUIRED（默认值）

在有transaction状态下执行；如当前没有transaction，则创建新的transaction；

这是最普通的状态，一般这是最开始的入口方法，采取这个方案。

实现在mysql中，就是这么一句：

```sql
2019-07-29T13:07:18.364347Z7482451 Connect	root@localhost on xxxdatabase using TCP/IP
2019-07-29T13:07:18.366558Z7482451 Query	SET NAMES utf8
2019-07-29T13:07:18.367043Z7482451 Query	SET autocommit=1
2019-07-29T13:07:18.367370Z7482451 Query	SET autocommit=0
```

本质是创建一个connection，并且设置autocommit=0。



#### SUPPORTS && NOT_SUPPORTED

这俩状态是随遇而安的典型，都不会抛异常，安安静静的执行。前者是有事务就用事务，没有就不用，不主动，不拒绝。后者是不管之前有没有，反正到我这里就没有

SUPPORTS：如当前有transaction，则在transaction状态下执行；如果当前没有transaction，在无transaction状态下执行；

NOT_SUPPORTED：在无transaction状态下执行；如果当前已有transaction，则将当前transaction挂起。

这个实现方式就是上文中说过的，重新开连接，并且设置一个autocommit=1。

#### MANDATORY && NEVER

这俩状态是强制的典型，不符合预期就抛异常

MANDATORY：必须在有transaction状态下执行，如果当前没有transaction，则抛出异常IllegalTransactionStateException；

NEVER：在无transaction状态下执行；如果当前已有transaction，则抛出异 IllegalTransactionStateException。

#### REQUIRES_NEW

创建新的transaction并执行；如果当前已有transaction，则将当前transaction挂起；

![](http://img.skydrift.cn/1564405699.png)

从图中可以看出，这是通过创建一个新的TCP连接实现的，本质上和外部的事务已经没有关系了。

这就导致下列的结论

1. 内部的方法和外部的不是一个事务，无法保障同时成功失败。
2. 内部的方法rollback，外部事务正常不会受到影响。

