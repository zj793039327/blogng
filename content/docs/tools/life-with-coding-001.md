---
title: 漫步编程路（1）
author: ""
date: 2024-03-01 21:45:14
tags:
  - learn
  - tech
draft: false
weight: 1
---
介绍在mac笔记本怎么使用键盘和鼠标比较舒服。
<!--more-->

## 序言
自从多年前开始有机会用mac进行开发，已经有一些年头了，在此写一些东西，权当留个记录，给自己看，也抛砖引玉。
## 1 mac电脑上到底怎么用键鼠比较舒服

刚上手mac的时候，其实比较懵，命令行啥的对我来说用的真不多。右上角也没有了关闭按钮，都到左上角去了。

### 痛点1：键盘的重复延迟高
![](http://img.skydrift.cn/pic_mac_coding_very_slow.gif)

如上图，其实还是挺难受的。甚至还不如自己多按几次。

这个问题其实比较容易解决，直接调整mac的设置即可，重复前延迟，按键重复速率调快，重复前延迟调低就好了。

![](http://img.skydrift.cn/1705381982.png?imageMogr2/thumbnail/!70p)

设置->键盘->重复速率，重复前延迟

调完以后的效果：


![](http://img.skydrift.cn/pic_mac_coding_after_adjust.gif?imageMogr2/thumbnail/!70p)

其实如果熟悉命令行的同学来说，一个命令就完事了：`ctrl + w`

![](http://img.skydrift.cn/pic_ctrl_w_can_fast_delete.gif?imageMogr2/thumbnail/!70p)

直接可以按照word进行删除的，这里的word，就是按照空格拆开的。

> 这也是其他程序员没有大肆的在网上讨论这个的原因，因为人家有更好的解决方案，我不知道而已。

知道了这个命令之后，不免想知道，是不是还有其他的命令行了。

1. 确实还有其他命令，很多文档说的非常详细[^1][^2][^3]，大家有兴趣可以搜一下。
2. `ctrl + w` 这个命令，在emacs里面也是一样的，**这莫非是一种巧合？**
3. 
其实emacs里面有很多快捷键，可以看下下图的操作。
![](http://img.skydrift.cn/pic_emacs_hot_keys.gif?imageMogr2/thumbnail/!70p)
### 痛点2：触控板如何进行拖动？

正常来讲，普通PC的话，有个鼠标，鼠标的操作很直观，比如拖动一个东西，可以直接左键按下，移动鼠标即可（可以简单的单手完成）但是在mac的话，如果只用触控版，那么就会有一些问题

1. 左键按下去拖动（一个指头操作，太费力了）
2. 一个指头按下去，另一个指头拖动（没办法拖的很远，两只手都需要离开位置）

这些操作都能实现，但是都不舒服。

> 我的感觉就是不太舒服，但是我在网上浏览，发现大家可能要求更高，比如双手不用挪动就实现鼠标的操作[^5]。
>
> 【**永远不要把你双手从字符键上移开**】这句口号给当时的我带来了不小的震撼
>
> > 好吧，真心话是不明觉厉，现在越看越有味道

那怎么办呢，其实mac提供了非常好的一个功能：三指拖移

> 这个功能是当时 勇哥 告诉我的，至今让我记记犹新，他也不多解释，就是说你用着就知道了

![](http://img.skydrift.cn/1705384321.png?imageMogr2/thumbnail/!70p)

设置 -> 辅助功能 -> 指针控制  -> 触控板 -> 使用触控板进行拖移 -> 三指拖移

设置完以后，三指拖移就是直接拖动当前屏幕了。

很顺溜，有没有？

### 痛点3：触控板 VS 键盘 & 鼠标

在上一个原则（以下简称第一原则）：【永远不要把你双手从字符键上移开】的基础上，其实这个问题答案非常明显了。

触控板肯定效率最高，因为双手不用离开。非常稳健

但是那我就喜欢噼里啪啦的感觉，或者就是有个支架，要把电脑放上去咋办？【嗯，说的是我】

| ![](http://img.skydrift.cn/1705385036.png?imageMogr2/thumbnail/!30p) | ![](http://img.skydrift.cn/1705385062.png?imageMogr2/thumbnail/!70p) |
| -------------------------------------------------------------------- | -------------------------------------------------------------------- |
但是用起来都不怎么样

![](http://img.skydrift.cn/1705385144.png?imageMogr2/thumbnail/!70p)

#### 方案1：机械键盘+无线鼠标

这个就很简单了，普通的机械键盘一枚，加上无线鼠标（或者蓝牙鼠标）就可以了

为啥要强调无线，因为有很多线的话，看起来太烦人了。无线技术也比较发达了，让自己省心点不好吗。

> apple的那个鼠标就别考虑了，太imba了，虽然支持部分手势，但是那个充电孔[^7]的设计，jobs看到了绝对会气的出来。
>
> ![](http://img.skydrift.cn/1705385367.png?imageMogr2/thumbnail/!70p)![](http://img.skydrift.cn/1705385377.png?imageMogr2/thumbnail/!70p)

#### 方案2：键盘+触控板

然而上面的方案，又回到原地了，感觉违反了第一原则的呢。让双手不离开

![](http://img.skydrift.cn/1705385518.png?imageMogr2/thumbnail/!70p)

发现有了这个trackpad，感觉也应该不错

![](http://img.skydrift.cn/1705385559.png?imageMogr2/thumbnail/!70p)

这样子是不是感觉也还挺好的？所以我直接去JD买了一个。试用了一天，直接退款了。

![](http://img.skydrift.cn/1705385673.png)

原因如下：

1. 太贵了
2. 和我的键盘是在太不搭配了。机械键盘的键程比较高，触控板比较薄，用起来的差异非常大，下图仅供参考

![](http://img.skydrift.cn/1705385875.png?imageMogr2/thumbnail/!30p)

用起来很不顺手，比起mac触控板浑然一体的感觉，其实还是差很多的。

#### 方案3：指点杆键盘

又搞了两天，感觉还是差点意思，怎么办呢？

这时候我想起来在驻地办公时候，有个老兄用thinkpad用的出神入化，最传神的莫过于红色的小点点用的好：

```
1. 右手食指触控小红点，做鼠标移动
2. 左手大拇指点击左键，作为鼠标左键点击
```

这两个操作，完美的满足了第一原则。

然后我在知乎也看到了很多有意思的回答，比如用小红点打游戏的[^6]，果然，男人对游戏的热情是真的厉害，记得有个老哥说在春运回家的火车上，用小红点打红警的。。。

想想其实完美符合第一原则

1. 由于特殊限定（春运火车）地方小，双手被迫只有小地方
2. 对游戏五笔的热情

> 关键是在知乎，有人提一些有趣的问题：
>
> ![](http://img.skydrift.cn/1705386624.png)
>
> 这个问题内容虽然看起来有点那个，但确实是个好问题，只有这样，才能引出来一帮同学一起讨论和分析[^8]
>
> 我也受益不少
>
> ![](http://img.skydrift.cn/1705386804.png)
>
> 这个兄弟的回答，也很发人深思，所有抛开使用环境提出的问题，其实都有点耍流氓。

虽然不是thinkpad，是否有独立的键盘呢？

![](http://img.skydrift.cn/1705386961.png)

确实有键盘，但是不便宜

有线的键盘：299：https://item.jd.com/68125647702.html

![](http://img.skydrift.cn/1705387168.png?imageMogr2/thumbnail/!70p)

无线的键盘：569：https://item.jd.com/10045399652864.html

![](http://img.skydrift.cn/1705387130.png?imageMogr2/thumbnail/!70p)

好，到现在，所有都搞定了，是时候生产力了~

什么？变形金刚黑苹果出了新款？

![](http://img.skydrift.cn/1705387391.png?imageMogr2/thumbnail/!70p)

## 2 磨刀不误砍柴工：vim的哲学

通过上面的发散启蒙，我发现了vim是个好东西，比起学习分散的编辑器，以及编辑器里面的快捷键，vim其实是硬通货。

为什么这么说？比如你看这篇[文章](https://www.zhihu.com/question/20783392/answer/27211385)[^5]，你会发现，很多编辑器中，都有vim的身影，idea，emacs，sublime，vscode，都有vim模式，甚至，连chrome都有vim模式

![](http://img.skydrift.cn/1705387580.png)

看来vim确实是很不错的，有这么多人推介，有这么多人背书，值得一学。Vim确实可以提高输入效率，用了vim之后，其实就不是输入单字，而是输出【段】了。到底Vim有何好处，如何学习？且听下回分解。


## x 参考文献



[^1]: https://gist.github.com/bradtraversy/cc180de0edee05075a6139e42d5f28ce linux命令行快捷键 一篇英文总结（非官方）
[^2]: https://gist.github.com/zhulianhua/befb8f61db8c72b4763d linux命令行快捷键  一篇中文总结（非官方）
[^3]: https://www.javatpoint.com/linux-terminal-shortcuts linux命令行快捷键 另一篇英文总结（非官方）

[^4]: https://www.zhihu.com/question/20783392/answer/27211385 如何成为 IntelliJ IDEA 键盘流？
[^5]: https://www.zhihu.com/question/39953134/answer/100980398 永远不要把你的双手从键盘挪开
[^6]: https://www.zhihu.com/question/322910621/answer/1934358069 小红点打游戏
[^7]: https://mrmad.com.tw/apple-magic-mouse-2-is-super-stupid 妙控鼠标的充电姿势
[^8]: https://www.zhihu.com/topic/20902866/hot thinkpad小红点
[^9]: https://zhuanlan.zhihu.com/p/582784251 说说ThinkPad TrackPoint Keyboard II小红点键盘 
