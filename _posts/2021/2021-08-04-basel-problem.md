---
layout: post
title: Basel Problem的几何含义  
categories: Math
description: Basel Problem的几何含义  
keywords: Math
mathjax: true
---

原作者: 3Blue1Brown

***  

# 1 序言
小时候学数学,第一次了解圆时,就被其简洁之美所吸引:一个变量(半径/直径)就可以定义一个圆  .
然而当我了解到圆周率π时,一种不"和谐"的感觉马上袭来.  
一个不知道从哪里蹦出来的无理数--进一步来说,其实是超越数(不能作为有理系数多项式方程的根的数)--比√2感觉还无厘头.  
√2通过一个简单的乘方运算即可得到"基本"的正整数,而π给我的感觉就是无迹可寻.  
至于高中接触到自然对数的底e时,更是丈二和尚摸不到头脑--一堆材料完全是循环定义--可见我国之前的数学人才水平有多么单薄,连一个能把问题讲清讲透的基础教材都没有.  
跑题了,收回来.然而长大后发现,其实π和e也没有那么难于理解.关于e的讲解可以看我转载的另一篇[文章](/2021/08/03/e/).今天我们通过Basel-Problem来重新了解一下π.

# 2 Basel Problem简介
以下摘自百度百科
```
巴塞尔问题是一个著名的级数问题，这个问题首先由皮耶特罗·门戈利在1644年提出，由莱昂哈德·欧拉在1735年解决。
但当时他的证明还不是十分严密，真正严密的证明在1741年给出。
由于这个问题难倒了以前许多的数学家，欧拉一解出这个问题马上就出名了，当时他二十八岁。
欧拉把这个问题作了一番推广，他的想法后来被黎曼在1859年的论文《论小于给定大数的素数个数》（On the Number of Primes Less Than a Given Magnitude）中所采用，论文中定义了黎曼ζ函数，并证明了它的一些基本的性质。
这个问题是以瑞士的第三大城市巴塞尔命名的，它是欧拉和伯努利家族的家乡。
```
看完不禁感慨,1735年西方的数学家就已经能解决这样的问题.而同年,写了《大义觉迷录》雍正帝驾崩,乾隆皇帝继位.  
又跑题了,收回来.Basel Problem是求下面级数的值.  
$$ \sum_{n=1}^\infty \frac{1}{n^2} = \lim_{n \to +\infty} (1+\frac{1}{2^2}+\frac{1}{3^2}+\cdots+\frac{1}{n^2}) $$  
欧拉解决并证明了,解是$$ \frac{\pi^2}{6} $$  

看完第一感觉,哇!π看起来不是那么"无厘头"了,而是很容易通过相对"基本"的计算得到,这让π看起来,"合理"了很多.

然而这个解该如何理解呢?下面让我们给出一个几何理解.

# 3 几何理解
## 3.1 转换为远近光源问题
观察公式,可以发现,里面包含一个平方反比的关系.  
学过中学物理的我们可以很容易联想起`牛顿万有引力定律`,`库伦定律`.  
从这个思路,我们可以假设有一个数轴,在每个正整数点上有一个相同的标准光源(亮度设定为1).  
![](/images/posts/2021/basel-problem/1.png)  
则Basel Problem可以转换为在原点的光线接收器接收的光线总亮度.  
这一步比较好理解,但是似乎什么也没做.  
别急,下面的思路是**在不改变原点光线接收器接收的总亮度(问题的解)的前提下,重新调整光源分布并赋予其某种易于理解的几何意义**.

## 3.2 勾股定理的其他形式
下面引入一个引理  
![](/images/posts/2021/basel-problem/2.png)  
上图中在原点接收到的原始灯塔(与原点距离为h)的亮度等于A,B两灯塔之和  
这是为什么呢?  
因为勾股定理的另一个形式如下  
$$ \frac{1}{h^2} = \frac{1}{a^2} + \frac{1}{b^2}$$  
证明如下  
$$ Area = \frac{1}{2}ab = \frac{1}{2}ch \quad \text{c是斜边长度} \\
a^2b^2=c^2h^2 \\
a^2b^2=(a^2+b^2)h^2 \\
\frac{1}{h^2} = \frac{1}{a^2} + \frac{1}{b^2}
$$  
如你所见,我们已经得到了一个方法,能够调整光源分布,我们离我们的目的又前进了一步.  

## 3.3 调整分布
![](/images/posts/2021/basel-problem/3.png)
如上图,假设在一个周长为2的圆的一条直径的两端分别有一个灯塔和一个光线接收器.  
易知  
$$ \text{直径长度为} \quad \frac{2}{\pi} \\
\text{则接收亮度为} \quad \frac{\pi^2}{4}
$$

## 3.3.1 第一次变换
![](/images/posts/2021/basel-problem/4.png)  
如上图,我们接着画一个周长为4的圆.根据章节[3.2](#32-勾股定理的其他形式),我们可以把原来的灯塔替换为新的圆上的两座灯塔.  
注意:**这里新的两座灯塔距离光线接收器的圆弧长度仍然为1**.  

## 3.3.2 第二次变换
![](/images/posts/2021/basel-problem/5.png)  
如上图,我们接着画一个周长为8的圆.  
我们连接新的圆的圆心和原来的两座灯塔,两条连接线在新的圆上有4个交点.  
我们在交点上放置新的灯塔.  
由此我们可以把原来的在周长为4的圆上的两座灯塔替换为新的圆上的4座灯塔.  
这是因为**圆直径所对的圆周角总是直角**,所以章节[3.2](#32-勾股定理的其他形式)成立.  
注意:**这里新的4座灯塔把周长8的圆四等分,每段弧长为2**.  

## 3.3.3 第三次变换
![](/images/posts/2021/basel-problem/6.png)  
如上图,我们接着画一个周长为16的圆.我们可以把原来的在周长为8的圆上的4座灯塔替换为新的圆上的8座灯塔.  
这里我们注意到规律,**新画的圆上的灯塔总是"均匀"的分布,总是把新画的圆n等分**.  
其实道理不难,我们注意到,新的圆的圆心总是在上一步操作的圆上,而且是在光线接收器所在直径的对面,再因为**圆周角=圆心角的一半**,所以每次操作实际是把上次的角度折半,那么既然开始是'均匀'的,后续的操作都是'均匀'的.  
这里我们注意到:**由于我们每次变换,圆的周长增加一倍,灯塔数也增加一倍.所以相邻灯塔间的圆弧长度不变,始终为2**.  

## 3.3.4 第∞次变换
![](/images/posts/2021/basel-problem/7.png)
如上图,想象我们无穷次的进行变换,圆变得无穷大.  
最后圆弧近似于一条直线.按照我们之前的发现,相邻灯塔的间隔是2, 可得  
$$ \cdots+\frac{1}{(-5)^2}+\frac{1}{(-3)^2}+\frac{1}{(-1)^2}+\frac{1}{1^2}+\frac{1}{3^2} +\frac{1}{5^2}+\cdots = \frac{\pi^2}{4}$$  
现在我们已经得到了一种通过'基本'运算来得到π的方法,但是我们还没有得到Basel Problem的解.  
不过这并不难,只需要一点简单的变化.  
$$
\cdots+\frac{1}{(-5)^2}+\frac{1}{(-3)^2}+\frac{1}{(-1)^2}+\frac{1}{1^2}+\frac{1}{3^2} +\frac{1}{5^2}+\cdots = \frac{\pi^2}{4} \\
\frac{1}{1^2}+\frac{1}{3^2} +\frac{1}{5^2}+\cdots= \frac{\pi^2}{8} \tag{1}
$$  

$$
\text{设x为Basel-Problem的解} \quad 1+\frac{1}{2^2}+\frac{1}{3^2}+\cdots = x \tag{2}
$$  
$$
\text{则易知} \quad \frac{1}{2^2}+\frac{1}{4^2}+\frac{1}{6^2}+\cdots = \frac{x}{4} \tag{3}
$$

$$
\text{(1)+(3)可得} \\
\frac{1}{1^2}+\frac{1}{2^2}+\frac{1}{3^2}+\frac{1}{4^2}+\frac{1}{5^2}+\cdots = \frac{\pi^2}{8} + \frac{x}{4} \tag{4}
$$  
$$
\text{将(2)代入(4)可得} \\
x = \frac{\pi^2}{8} + \frac{x}{4} \\
\frac{3x}{4} = \frac{\pi^2}{8} \\
x = \frac{\pi^2}{6}
$$

# 4 感想
小的时候,听过一个曹冲称象的故事.  
曹冲小小年纪就如此聪明,令人印象深刻.不过我理解曹冲称象关键是包含了人类的两大智慧.  

1. **把一个未知问题转换为另外一个等价的问题**.  
2. **把转换的问题,分解为很多规模较小,易于解决的问题.最后再把子问题的解组合起来得到父问题的解**.  

曹冲先把大象的重量转换为一船石头的重量  
然后通过计算一堆堆石头的重量和来得到大象的重量  

本问题也是一样  
我们先把问题转换为光源的问题  
然后给了光源问题几何意义,得到一个易于理解的形式  
最后通过简单的变换即可得到basel问题的解  
