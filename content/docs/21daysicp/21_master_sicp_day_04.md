---
title : "Day 4 深入递归02"
author : ["zj56"]
draft : true
weight : 4
---


{{< katex display=true class="optional" >}}

\begin{flalign}
&n轮: f(5,3)\\
=& f(5,2) + f(5-5=0,3) \\
& \textcolor{grey}{;;f(5-5,3) :\quad 因为第3种硬币的面额是5元}\\
=& f(5,2) + f(0,3) \\
&\textcolor{grey}{;;[3种硬币(1,2,5)组合5块] = [2种硬币(1,2)组合5块] + [3种硬币(1,2,5)组合0元]}\\
=& f(5,2) + 1 \\
&\textcolor{grey}{;;f(0,3)：此处需要计算这个值，对于amount的解释分成3种情况}\\
& \textcolor{green} {;;\quad amount 
\left\{
 \begin{array}{lrc}
  =0,\quad 此时代表找到了方式，因为硬币的面额正好和金额整除了 \\
  <0,\quad 此时代表没找到方式，零钱的面额比剩余的总额要大了\\
  >0,\quad 此时代表需要继续进行处理 \\
 \end{array}
\right \}} \\
&\textcolor{grey}{;;f(0,3) 中，amount=0，因此找到一种方式，返回1，即f(0,n)=1 } \\
&\textcolor{grey}{;;[3种硬币(1,2,5)组合5块] = [2种硬币(1,2)组合5块] + 1} \\
&&
\end{flalign}

{{< /katex >}}
##  评估递归对资源的消耗 {#评估递归对资源的消耗}


##  降低递归时候资源消耗的办法 {#降低递归时候资源消耗的办法}


### 求幂过程引出的点 {#求幂过程引出的点}


### 最大公约数过程引出的点 {#最大公约数过程引出的点}

asdasd
