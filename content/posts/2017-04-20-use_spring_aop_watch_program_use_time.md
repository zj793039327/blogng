---
title: 使用 spring AOP 监控代码的执行耗时
date: 2017-04-20 16:49:46
modified: 2017-04-20 16:49:46
author: jojoster
postid: 205
slug: 205
nicename: use_spring_aop_watch_program_use_time
attachments: $ATTACHMENTS
layout: post
tags:
  - java
categories: tech
---

使用spring提供的aop功能，我们可以很方便的实现动态代理的功能。在使用上，spring提供了两种不同的实现，分别是Spring AOP 和 AspectJ。
<!--more-->

# 前言

使用spring提供的aop功能，我们可以很方便的实现动态代理的功能。在使用上，spring提供了两种不同的实现，分别是Spring AOP 和 AspectJ。

# 提供方式以及对比

## spring AOP

### 概述

1. 纯java实现，不需要额外的编译流程，不需要引用其他三方包。
2. 适合集成到Servlet容器或者应用服务中
3. 仅支持方法级别的代理，不支持成员变量
4. 设计宗旨是集成IoC，并且有效的解决大部分企业级应用中的需求，不同于AspectJ的细粒度。
5. 与AspectJ进行互补
6. ​

> 非侵略性是spring设计的一个中心原则，一般情况下，不会有spring的代码存在与业务代码中。但是某些情况中，不是这样，比如注解。

### 实现

默认使用标准JDK中的 动态代理是实现，针对任意接口，都可以实现。

可以配置使用CGLIB进行代理，可以针对类进行代理。CGLIB的使用，对开发者是透明得到，在针对没有实现接口的类进行代理的时候，spring会自动使用CGLIB进行实现。

一般情况下，建议业务类都实现一个接口，是比较好的编程实现

> 这个另说把，不必要的接口实现了以后，除了繁琐没有其他作用

在一个类实现了多个接口的时候，可以强制使用CGLIB。

>很多考虑的地方 [spring文档](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/aop.html#aop-proxying)
> ```xml
> <aop:config proxy-target-class="true">
>     <!-- other beans defined here... -->
> </aop:config>
> ```

# 案例

项目需要统计一下一个service类的核心方法执行时间

但是通常，一个核心方法内部有很多的子方法，如何在做代理的时候，将所所有方法进行代理

本次的实现，使用了AspectJ。

代码如下：

### 代理配置

```java
package com.xxx.utils;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

/**
 * Created by xxxx on 17/3/22.
 */
@Aspect
@Component
public class CostTimeAspect {

    @Around("execution(* com.xxx.service.AService.*(..))")
    public Object printTimeMethod(ProceedingJoinPoint pjp) throws Throwable {

        long time = System.currentTimeMillis();
        Object obj = pjp.proceed();
        long cost = System.currentTimeMillis() - time;
        if (cost > 0) {
            System.out.println(pjp.getSignature().toShortString() + "costs mills:" + cost);
        }
        return obj;
    }

}

```

### spring.xml

```xml
<aop:aspectj-autoproxy />
```

### pom.xml

需要在 build 节点中增加，并且要适配指定的jdk版本

```xml
<plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>aspectj-maven-plugin</artifactId>
            <version>1.4</version>
            <dependencies>
                <dependency>
                    <groupId>org.aspectj</groupId>
                    <artifactId>aspectjrt</artifactId>
                    <version>1.7.3</version>
                </dependency>
                <dependency>
                    <groupId>org.aspectj</groupId>
                    <artifactId>aspectjtools</artifactId>
                    <version>1.7.3</version>
                </dependency>
            </dependencies>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>test-compile</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <outxml>true</outxml>
                <verbose>true</verbose>
                <showWeaveInfo>true</showWeaveInfo>
                <aspectLibraries>
                    <aspectLibrary>
                        <groupId>org.springframework</groupId>
                        <artifactId>spring-aspects</artifactId>
                    </aspectLibrary>
                </aspectLibraries>
                <source>1.6</source>
                <target>1.6</target>
            </configuration>
        </plugin>

```

