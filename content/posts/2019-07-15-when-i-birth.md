---
layout: post
title:  "hello world"
categories: nothing
tags: readme demo markdown
author: j56
comments: false # true 则开启评论
mathjax: true # true 则开启加载数学公式
---

## 引入

这是一篇helloworld，主要记录一下开始折腾的时间。
<!--more-->

写一些鸡汤样式的文字，什么`每天前进一点点`等等，其实都是瞎说，多说无益。

## markdown格式

本博客中，渲染markdown的引擎是 kramdown， 具体语法参见 [这里](https://kramdown.gettalong.org/syntax.html)。

当然markdown最基础的语法请参照[这里](https://www.appinn.com/markdown/)。

这里着重介绍下markdown的格式，参考：[wiki百科](https://zh.wikipedia.org/wiki/Markdown)。

实际的编辑器，在MAC推荐使用 typora，很多时候，挺好用的。







# 这是1号标题&目录
这底下都是标题的，所有的标题都会自动进入到目录中，启用目录参见文章头部

## 2号标题

### 3号标题

#### 4号标题

##### 5号标题

###### 6号标题

## 文字的表现形式

我是一段示例文字[^1]，有**粗体**，*斜体*，<u>下划线</u>，`内联代码`，~~删除线~~，<!--注释-->

[超链接](http://www.baidu.com)


### 文内链接
这是一个文内链接的[例子](http://example.com/ "鼠标悬浮此处显示的标题")。

[这个](http://example.net/)链接在鼠标悬浮时没有标题。

[这个](/about/)链接是本地资源。

### 引用链接

这是一个引用链接的[例子][id]。

[id]: http://example.com/  "鼠标悬浮标题（可选）"

### 注意，这里的id没有大小写区分，如果省略id，则前面方括号的内容会被用作id。

我常用的网站包括[Google][1]，[Yahoo][2]和[MSN][3]。

[1]: http://google.com/        "Google"
[2]: http://search.yahoo.com/  "Yahoo Search"
[3]: http://search.msn.com/    "MSN Search"

### 也可以写成

我常用的网站包括[Google][]，[Yahoo][]和[MSN][]。

[google]: http://google.com/        "Google"
[yahoo]:  http://search.yahoo.com/  "Yahoo Search"
[msn]:    http://search.msn.com/    "MSN Search"

![我是百度主页图片](https://www.baidu.com/img/bd_logo1.png?where=super)

无序列表

* 1
* 2
* 3


有序列表

1. 1
2. 2
3. 3

> 我是一段引用，很好看是不是
>
> 引用内部啥都能写
>
> 我是一段示例文字，有**粗体**，*斜体*，<u>下划线</u>，`内联代码`，~~删除线~~，<!--注释-->

各类分割线

* * *

***

*****

- - -

---------------------------------------



## 专业表现形式

### 表格

|                                 | 2000   | 5000   | 8000   | 10000  | 正负样本悬殊 |
| ------------------------------- | ------ | ------ | ------ | ------ | ------ |
| DecisionTreeClassifier(gini)    | 98.15% | 99%    | 99.13% | 99.31% | 99.94% |
| DecisionTreeClassifier(entropy) | 98.6%  | 99.28% | 99.55% | 99.53% | 99.96% |
| RandomForestClassifier          | 98.3%  | 99.03% | 99.33% | 99.46% | 99.93% |
| GradientBoostingClassifier      | 99.3%  | 99.45% | 99.51% | 99.57% | 99.64% |

### 代码

```java
import java.lang.String;
```

### 数学图表

集合信息的度量方式称为香农熵，简称熵（entropy）。

熵就是信息的期望值。单位是"比特"（bit）。一个比特是一位二进制数字。比如 "将一个硬币朝天空抛起4次，最后四次落地时候的朝向" 这句话中包含一定的信息量。最终答案需要我们猜测4次，每次猜一次硬币落地的结果，猜测结果只有2种，分别是正面和反面。每个面朝向的概率是1/2。

如果一次关于1/2猜测的信息量=1，那么上面一句话的信息量=4，比2更少的情况就是1，不需要猜测，那么信息量就是0。

同理，如果向上抛起的是一个骰子，每次猜测的结果有6种，每种结果出现的概率是1/6，每次猜测的次数大约=2.5（类似折半查找）。那么上面语句信息量大约是2.5*4=10

引入对数的话，每次猜测可以表示为：

$$
l(x_i)=-log_2p(x_i)
$$

其中，p代表发生x事件的概率，x代表事件，i代表事件的结果

熵的公式则如下

$$
Ent=-\sum_{i=1}^{n}p(x_i)log_2p(x_i)
$$

其中n是分类的数目，注意，在最终的熵需要乘以事件的概率，对于上面硬币的例子

$$
Ent=-4\times(\frac{1}{2}\times log_2\frac12)=2(bit)
$$

其中，p代表发生x事件的概率，x代表事件，i代表事件的结果



[^1]:  我是脚注，可以是一些引用，或者一些超链接，或者是图片地址等