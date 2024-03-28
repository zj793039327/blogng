---
title: "Day 3 深入递归"
author: ["joe567"]
draft: true
weight: 3
tags: sicp,emacs
params:
  math: true
---

## 1 树形递归

计算斐波那契数列的图形，可以看做是一个树形递归

{{< figure src="http://img.skydrift.cn/1708768655.png?imageMogr2/thumbnail/!70p" >}}

斐波那契数列，很简单可以用scheme实现

```
(define (fib n)
   (cond ((= n 0) 0)
         ((= n 1) 1)
         (else (+ (fib (- n 1))
                  (fib (- n 2))))))
```

可以看到，这个表达式明显是递归型的，因为会有结构的展开，有 `+` 运算符在 `fib` 的外面，因此后者阻碍了前者的计算，这个也会进行展开

展开的结构和之前不一样，比较复杂。这个例子是展现树递归的好案例，但是在计算fib的值方面，显然效率很低，因为有很多很多的无效计算。

{{< figure src="http://img.skydrift.cn/1708769038.png?imageMogr2/thumbnail/!70p" >}}

可以用线性递归的形式进行改写，Fibonacci数列本身其实就是 `a+b` ，只要不断的代换

1. 第一轮：a1=0，b1=1
2. 第二轮：a2=1（a1+b1），b2=1(a1)
3. 第三轮：a3=2（a2+b2），b3=1(a2)
4. 第四轮：a4=3(a3+b3), b4=2(a3)
5. 第五轮：a5=5(a4+b4),b5=3(a4)

<!--listend-->

```scheme
(define (fib n) (fib-iter 1 0 n))
(define (fib-iter a b count)
     (if (= count 0)
      b
     (fib-iter (+ a b) a (- count 1))))
```

树形递归看上去效率非常低，但是非常直观，在一定规模内，解决问题的效率非常高

## 2 树形递归的案例：找零钱

比如这问题：100元人民币，换成零钱，1块的，5块的，10块的，20块的这几种，一共有几种算法？

如果按照递归考虑，那么这问题可以简单化

1. 找出来 a 元的 零钱总数 n =
2. 将现金换成 除了 第一种零钱 之外的 其他所有零钱 的不同方式数目 +
3. 将现金 a - d 换成所有种类的零钱 的不同方式的数目 ，其中 d 是一种零钱的面额

思路是自然解题的倒推方式

$$
假设 f(amount\_n,coin\_j) = 针对特定金额，使用 j 种硬币的找零方式 ,  其中，n代表迭代的轮次，j代表包含j种类的硬币
$$

公式相对还是比较好理解的，代表的是最终的状态，目标是进行递归推导，考虑到数学归纳法，那么这个过程可以拆解成如下的部分，并且进行迭代

$$
第n轮，f(amount\_n, coin\_j) = f(amount\_n, coin\_{j-1}) + f(amount\_{n-1}, coin\_j)
$$


如何理解这个递归拆分呢，用实例举例来帮助理解，比如我们要计算

$$
f(5,3),当 coins=[1,2,5]
$$

解释如下：要找总额 `5` 块的零钱，用了 `3` 种面额的硬币，分别是 `[1,2,5]` 代表 `1` 块， `2 块， =5` 块的

> 其实就是示例1，表达式不严谨，将就看下
>
> 1. amount_n 代表第n轮中的总金额，在n轮，也就是5元
> 2. coin_j 代表此时用3种硬币进行组合，在n轮，也就是3种硬币

用实例来表示如下

$$
\begin{flalign}
&n轮: f(5,3)\\\\
=& f(5,2) + f(5-5=0,3) \\\\
& \textcolor{grey}{;;f(5-5,3) :\quad 因为第3种硬币的面额是5元}\\\\
=& f(5,2) + f(0,3) \\\\
&\textcolor{grey}{;;[3种硬币(1,2,5)组合5块] = [2种硬币(1,2)组合5块] + [3种硬币(1,2,5)组合0元]}\\\\
=& f(5,2) + 1 \\\\
&\textcolor{grey}{;;f(0,3)：此处需要计算这个值，对于amount的解释分成3种情况}\\\\
& \textcolor{green} {;;\quad amount
\left\\{
\begin{array}{lrc}
=0,\quad 此时代表找到了方式，因为硬币的面额正好和金额整除了 \\\\
<0,\quad 此时代表没找到方式，零钱的面额比剩余的总额要大了\\\\
>0,\quad 此时代表需要继续进行处理 \\\\
\end{array}
\right\\}} \\\\
&\textcolor{grey}{;;f(0,3) 中，amount=0，因此找到一种方式，返回1，即f(0,n)=1 } \\\\
&\textcolor{grey}{;;[3种硬币(1,2,5)组合5块] = [2种硬币(1,2)组合5块] + 1} \\\\
&&
\end{flalign}
$$

$$
\require{ams}
\begin{flalign}
&n-1 到 1轮：f(5,2)\\\\
&= f(5,1)+f(3,2) \quad\quad\quad\textcolor{grey}{;; f(3,2) = f(5-2,2)}\\\\
&= [f(5,0) + f(4,1)] + [f(3,1) + f(1,2)]\quad\quad\quad\textcolor{grey}{;; f(4,1) = f(5-1,1), f(1,2) = f(3-2,1)}\\\\
& \textcolor{grey}{;;\quad对于计算 硬币的数量=0的场景，自然而然，可用的方式=0，因此f (n,0)=0} \\\\
&= [0 + f(4,1)] + [f(3,1) + f(1,2)] \\\\
&= \\{0 + [f(4,0) + f(3,1)]\\} + \\{[f(2,1) + f(2,0)] + [f(0,2) + f(1,1)]\\} \\\\
&= \\{0 + [0 + f(3,1)]\\} + \\{[f(2,1) + 0] + [0 + \textcolor{red}{f(1,1)}]\\} \\\\
&= f(3,1) + f(2,1) + \textcolor{red}{f(1,1)} \\\\
&= [f(2,1) + f(0,1)] + [\textcolor{red}{f(1,1)}+f(1,0)] + [f(0,1)+f(1,0)] \\\\
&= [f(2,1) + 0] + [f(1,1)+ 0] + [1+0] \\\\
&= [\textcolor{red}{f(1,1)}+f(1,0)] + [f(0,1)+f(1,0)] + 1 \\\\
&= \textcolor{red}{f(1,1)} + 1 + 1 \\\\
&= [f(0,1)+f(1,0)] + 1 + 1 \\\\
&= 1 + 1 + 1 \\\\
&= 3 \\\\
&&
\end{flalign}
$$

计算结果 \\(f(5,3)=4,当 coins=[1,2,5]\\)，可以看到，这里面有很多的冗余计算，仅仅一个 \\(f(1,1)\\) 就重复计算了很多次

因此，这种计算虽然可以给出正确的答案，但是效率比较低。

```scheme
(define (change amount coins)
  (-> exact-integer? (listof exact-integer?) exact-integer?)
  (define (cc amount kind-of-coins)
    (cond [(= amount 0) 1]
          [(or (< amount 0) (= kind-of-coins 0)) 0]
          [else (+
                  (cc amount (- kind-of-coins 1))
                  (cc (- amount (list-ref coins (- kind-of-coins 1))) kind-of-coins)
                )
           ]
    ))
    (cc amount (length coins))
  )
```

### 2.1 优化1：使用备忘录方式提升效率

```scheme
(define (change amount coins)
  (-> exact-integer? (listof exact-integer?) exact-integer?)
  (define tb (make-hash))
  (define (del1 x) (- x 1))
  (define (cc amount kind-of-coins)
    (cond [(= amount 0) 1]
          [(or (< amount 0) (= kind-of-coins 0)) 0]
          [else
           (define k1 (format "~s-~s" amount (del1 kind-of-coins)))
           (define k2 (format "~s-~s" (- amount (list-ref coins (del1 kind-of-coins))) kind-of-coins))
           (+
            (cond [(hash-has-key? tb k1)
                   (hash-ref tb k1)]
                  [else
                   (hash-set! tb k1  (cc amount (del1 kind-of-coins)))
                   (hash-ref tb k1)
                   ]
                  )
            (cond [(hash-has-key? tb k2)
                   (hash-ref tb k2)]
                  [else
                   (hash-set! tb k2 (cc (- amount (list-ref coins (del1 kind-of-coins))) kind-of-coins))
                   (hash-ref tb k2)
                   ]
                  )
            )
           ]
          ))
  (cc amount (length coins))
  )
```

使用了这个算法之后，很多计算过一次的 \\(f(a,b)\\) 可以不用重复计算，效率提高很多。

### 2.2 优化2：使用dp 动态规划解决问题

动态规划这个概念十分高大上，但是通过学习了解的话，感觉就是纸老虎。

正常的递归，是从 n 到 n-1 一直到 1
动态规划反其道行之，干脆从 1一直计算到 n，这样理解比较容易

[算法-动态规划 Dynamic Programming--从菜鸟到老鸟](https://blog.csdn.net/u013309870/article/details/75193592)[^fn:1]

[算法随想录](https://programmercarl.com/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html)[^fn:2]

下面是一段来自编程随想录的动态规划介绍，我感觉一语中的

> 动态规划，英文：Dynamic Programming，简称DP，如果某一问题有很多重叠子问题，使用动态规划是最有效的。
> 所以动态规划中每一个状态一定是由上一个状态推导出来的，\*\*这一点就区分于贪心\*\*，贪心没有状态推导，而是从局部直接选最优的，
>
> 在[关于贪心算法，你该了解这些！](https://programmercarl.com/%E8%B4%AA%E5%BF%83%E7%AE%97%E6%B3%95%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html)中我举了一个背包问题的例子。
>
> 例如：有N件物品和一个最多能背重量为W 的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。\*\*每件物品只能用一次\*\*，求解将哪些物品装入背包里物品价值总和最大。
> 动态规划中dp[j]是由dp[j-weight[i]]推导出来的，然后取max(dp[j], dp[j - weight[i]] + value[i])。
>
> 但如果是贪心呢，每次拿物品选一个最大的或者最小的就完事了，和上一个状态没有关系。
> 所以贪心解决不了动态规划的问题。
> 其实大家也不用死扣动规和贪心的理论区别，后面做做题目自然就知道了。
> 而且很多讲解动态规划的文章都会讲最优子结构啊和重叠子问题啊这些，这些东西都是教科书的上定义，晦涩难懂而且不实用。
> 大家知道动规是由前一个状态推导出来的，而贪心是局部直接选最优的，对于刷题来说就够用了。

#### 2.2.1 理解动态规划

没办法，我就是这么个死脑筋，想不明白就想要想明白，想不明白就很难受

并且必须非常直观才能想明白，说白了就是抽象思维差劲


最近又阅读了动态规划的一些文章，突然发现，这个思路貌似是符合动态规划的。

简言之，就是 `从下到上，通过空间换时间，其实是一种备忘录算法` 。

这个迭代写法，从底层说，就是上面一句话的最简实现。

1. `f-iter` 中的3个参数，其实就是动态规划中的 `d[]` 数组
2. 由于 `f-iter` 函数本身 有4个入参，相当于是 `d[0]~d[3]`
3. 由于 `f-iter` 本身逻辑较为简单，因此只需要 d[n-1], d[n-2], d[n-3] 即可，在上面的 `d[0]~d[2]` 中，循环设置了这3个值，即空间复杂度是 o(3)

为了方便理解，这样解释一下, 参见下面的表格

| f(n) 's n | f(n) 's value         | f-iter 's param | d array           |
| --------- | --------------------- | --------------- | ----------------- |
| 2         | 2                     | 2 1 0           | f(2) f(1) f(0)  0 |
| 3         | 2+2\*2           4    | 4 2 1           | f(3) f(2) f(1)  1 |
| 4         | 4+2\*2+3\*1       11  | 11 4 2          | f(4) f(3) f(2)  2 |
| 5         | 11+2\*4+3\*2      25  | 25 11 4         | f(5) f(4) f(2)  3 |
| 6         | 25+2\*11+3\*4     59  | 59 25 11        | f(6) f(5) f(4)  4 |
| 7         | 59+2\*25+3\*11    142 | 142 59 25       | f(7) f(6) f(5)  5 |

你会发现，f-iter中的所有参数，真真实实的是把f(n) 中需要的val存储起来了，这就是没有数组的存储

## 3 找零钱的算法复杂度

这是一个标准的递归树形结构算法，
时间复杂度和空间复杂度的计算，都有逻辑，通过一点点的进行 `代换` 可以比较好的掌握
具体的讨论，参见本人练习题 1.14的解答

### 3.1 空间复杂度

递归算法中，就是递归的调用层次，本题中，就是 S(n,m)=&Theta;(n)

### 3.2 时间复杂度

时间复杂度的计算，最初摸不着头脑，但是可以分解进行计算，比如先算 f(n,1)的，然后计算 f(n,2)的

以此类推
最终，时间复杂度，应该是 $T(n,m) = &Theta;(n^m)$


[^fn:1]: HankingHu的csdn博客 [算法-动态规划 Dynamic Programming--从菜鸟到老鸟](https://blog.csdn.net/u013309870/article/details/75193592)

[^fn:2]: 专注算法编程的carl [算法随想录](https://programmercarl.com/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html)
