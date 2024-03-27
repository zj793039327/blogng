---
layout: post
title:  "如何写一篇博文"
categories: tech
tags: writing
author: j56
comments: false # true 则开启评论
---

## 引入

这里主要看如何写一篇博文，主要是基于这个博客去实际的看如何写博文。
<!--more-->






## 写文章之前

### 获取库以及目录解析

首先需要对github的库进行folk。folk之后，应该可以看到自己的github有这个库。如下图

![我folk了这个项目](http://img.skydrift.cn/1563182133.png?imageMogr2/thumbnail/!70p)

folk之后，项目就完全的拷贝了一份到你自己的git仓库中。可以检出来了。

检出之后，可以看到目录如下，这是一个标准的 [jekyll博客](http://www.jekyllrb.com) 格式：

```
├── LICENSE
├── README.md
├── _config.yml
├── _includes
│   ├── ...
│   └── a.html
├── _layouts
│   ├── ...
│   └── post.html
├── _posts
│   ├── 2019-07-15-article1.md
│   └── 2019-07-16-article1.md
├── _sass
│   ├── ...
│   ├── _scrollbar.scss
│   └── _syntax-highlighting.scss
├── css
│   └── main.scss
├── favicon.ico
├── feed.xml
├── index.html
├── js
│   ├── ...
│   └── waterfall.js
└── page
└── ├── ...
    └── about.md
```

目录中，`_posts` 文件夹是存放文章的地方，文章同一采用md格式来编写。

### 文章命名规范

文章的命名规范如下：

yyyy-MM-dd-{name}.md

注意，这里的name可以使用可读性良好的英文字符来命名，不要使用驼峰和下划线，而是使用连字符，比如这篇文章，名字叫做

`2019-07-15-how-to-write-an-article.md`，jeykll会自动转化为：

`/2019/07/15/how-to-write-an-article/`

当然这个规则可以转化，在jekyll中都可以配置。

ps：文章的名称可以直接带空格，jekyll会自动转化为连字符，**但是**，这种路径在操作系统中不是很友好，mac和windows都不友好，尤其是在命令行下。

## 写文章

每一篇文章，都是一个markdown结构的文件，至于支持的内容特性，可以参见上一篇[博文](https://dulittlebylittle.github.io/2019/07/15/when-i-birth/)。

这里说一些博客定制的东西。

### 文章meta信息

每篇文章的title，都有一个这样的栏，这也是jekyll的规范。主要用来存储一些文章的meta信息，比如分类，标签啥的。

内容详细说明

```
---
layout: post # 标识这是一篇文章，同类型的还有pages等
title:  "如何写一篇博文" # 标题，务必用双引号标注
categories: tech # 目录
tags: writing # 标签
author: j56 # 作者
comments: false # 是否开启评论
mathjax: true # 是否开启数学公式渲染
---
```

categories的作用：可以标识和检索，每篇文章最好只有一个categories。

![](http://img.skydrift.cn/1563183366.png?imageMogr2/thumbnail/!70p)

ps：是不是有多个categories，我其实不知道。

tags：主要标识文章的内容，同类文章越多，标签就会越大。每篇文章可以有多个tag。

![](http://img.skydrift.cn/1563183556.png)

### Read All

![](http://img.skydrift.cn/1563183386.png?imageMogr2/thumbnail/!70p)

一篇文章如果字数很长，那么在首页以及列表中可以截断展示，后续部分通过点击Read All进入详情查看。

实际的编写中，只要空出三个空行，空行之前的内容会展示在首页，空行之后的内容则会隐藏不展示。

![](http://img.skydrift.cn/1563183407.png?imageMogr2/thumbnail/!70p)



### 上传图片

一张合适的好图，胜过千言万语，所以这个博客中，一定需要有一些展示良好的图片。

关于图片的展示，在markdown中其实很好做，方法有两种

1. 使用外部图床，比如7牛、阿里云等外部图床；
2. 直接把图片存储在boke路径之下。

两种方案各有各的好处，下面详细说下实现方案

#### 外部图床方案

![外部图床](http://img.skydrift.cn/1563190649.png)

看下地址哦

我这里找的是7牛云的图床，这个图床的好处就在于图片完全与博客分离。坏处也是这里，后续如果云服务中断，会造成图片没办法访问。

如果使用mac+alfred，那么有个十分 [爽快的方案](https://github.com/kaito-kidd/markdown-image-alfred)。

拿走不谢。



#### 内部图床方案

![image-20190715193815263](/img/2019-07-15-how-to-write-an-article-02.png)

看下地址哦（这是蓝猫，明明英短更可爱的。。。）

内部图床一般就是截图，然后保存成本地文件，然后一起提交。

具体可以参考下 [知乎的回答](https://www.zhihu.com/question/31123165)。

### 找一个好的markdown编辑器

mac中最好的markdown编辑器，鄙人觉得叫做 Typora。其它的都不如这个好用。

## 预览

对于已经写好的文章，肯定在本地发布并且预览是最好的，否则上了服务器看费力不讨好。重新提交又不是很方便。

由于jekyll本身是ruby实现的，当前的mac又都带了ruby的基本环境。搭建一套环境也很是方便。

百度一搜一大把，大家自行参考这篇[文章](https://www.jianshu.com/p/07064eb79740)吧。



## 提交并且发布

><div><font color='red'>注意1：之所以使用这样的提交策略，就是要让大家注意，别把你们项目中的隐私也一并提交，首先自己要保证一次，其次，博主也会有对应的审查。</font></div>
>
><font color='darkred'>注意2：写文章请原创，如果不是原创，请标明转载或或者征得作者同意再转载，如果有被侵权的朋友，也可以通过邮件反馈。</font><dulittlebylittle@yeah.net>

-----

对于已经写好的文章，如何提交到本博客中呢？由于已经首先folk了，这里的提交其实就是pull request了。简要的操作流程如下，我一步一步截图看下：

1. 提交到自己的github中，并且push。

   ![首先自己提交并且push](http://img.skydrift.cn/1563192851.png?imageMogr2/thumbnail/!70p)

2. 发起pull request。

   1. 首先是点击提交，本质这就是一个merge的过程，首先要更新到最新版本，然后增加一个新的提交。
   ![点击这里发起request](http://img.skydrift.cn/1563192891.png?imageMogr2/thumbnail/!70p)
   1. 提交之后是这个样子的。可以看到有一个open的request，博客的管理员可以看到，如果审核通过之后，可以合并并且公开。![提交之后是这样子的](http://img.skydrift.cn/1563193025.png?imageMogr2/thumbnail/!70p)
   1. 管理员可以在自己的界面看到这里：![一个Pull requests](http://img.skydrift.cn/1563193211.png?imageMogr2/thumbnail/!70p)

3. 等待博客管理员同意。![一个完整的通过了的request](http://img.skydrift.cn/1563194404.png)

是不是很简单。

当然本博客并不是十分开放，不是所有人提交都会合入的，哼。