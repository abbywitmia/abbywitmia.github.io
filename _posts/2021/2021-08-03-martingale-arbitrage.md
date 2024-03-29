---
layout: post
title: Martingale Arbitrage简介  
categories: Math
description: Martingale Arbitrage简介  
keywords: Math
mathjax: true
---

# 1 策略
一个赌场设有“猜大小”的赌局，玩家下注后猜“大”或者猜“小”，如果输了，则失去赌注，如果赢的话，则获得本金以及一倍的利润。 “必胜法”是这么玩的：  

（1）押100猜“大”；  
（2）如果赢的话，返回（1）继续；  
（3）如果输的话则将赌注翻倍后继续猜“大”，因为不可能连续出现“大”，总会有获胜的时候，而且由于赌注一直是翻倍，只要赢一次，就会把所有输掉的钱赢回；  
（4）只要赢了，就继续返回（1）。  

# 2 实验
不妨先赌一把试试：  
每局押 100 元，发哥带着 50000 的本金入场。  
假设每局玩 1 分钟，那么每小时 60 次，每天可以玩 360 次。  
那么第一天过去呢，看下发哥战绩如何呢？  
![](/images/posts/2021/martingale/1.png)  
天亮时，手里的钱已经有 67500 了，日入一万七，发哥觉得自己似乎找到了生命的真谛，一张丑脸上不禁泛起了微笑。  

哦对了，为了让我们的实验更加严谨，要先商量好，如果万一连续赌输，资金不够支持下一次赌注的时候，采取什么策略。  

这里采取的策略是：  
`下次赌注为全部余下资金，如果赌赢，那么回到（1）押100猜「大」`  
采取这种策略的原因是……这样编程比较方便。  

转眼又是一天过去了，来看下发哥今天战绩如何吧？  

咦，发哥你怎么不到天亮就回来了？  
什么，输光光？！  
到底发生了什么事？！  
![](/images/posts/2021/martingale/2.png)  
可以看到，从第 676 次时，发哥连续遇到了 10 次「小」， 好不容易积攒起来的 8 万财富毁于一旦。  

此处应该有掌声。  

痛定思痛，在排除了「赌场出老千」的因素以后，发哥总归还是不太甘心的……毕竟连续掷出 10 个「小」实在是很少见。  

由于本宇宙的资金已经耗尽，那么我们就派出平行宇宙的发哥再战一百回合吧！  

派出 100 个发哥荒野求生。  

20 年以后， 100 个中，能坚持自己梦想的有几人呢？  

结果出来了，先给众看官展示一下个人财富 Top 5 吧：  
![](/images/posts/2021/martingale/3.png)  

然后放出大饼图：  
![](/images/posts/2021/martingale/4.png)  
![](/images/posts/2021/martingale/5.png)  

如你所见，60% 的赌徒会在 5 天左右输光光，而半个月过后，还在继续的赌徒只剩 17% ，如果你足够努力，闯入三甲，那你账面上最多时会超过 100 万资金。  

前三甲以外的朋友们，在 50 天左右全部申请破产。  

另外，不得不提一下第一名这种不世出的天才，竟然坚持了 800 多天，巅峰期达到上千万，疯狂玷污实验数据……  

咳咳，所以说，实验结论就是，在统计数据明显不够充足的情况下（=_=），97% 的玩家在 50 天左右就输光，但与此同时，想依靠此路挣个上千万也不是没有可能的。  

另外，别忘了这是建立在输赢五五开、没有人出千、以及赌资不设上下限的情况下的。  

# 3 分析  
实验做完了，下面开始分析：  

发哥初始资金带了 50000，每次押注 100，为了方便一点我们假设他带了51200 入场。  

这样，在 51200 ~ 102300 这个区间，如果连续掷 9 次「小」，就还能剩下 100 ~ 51100 。也就是说，虽然连续 10 次「小」才会输光，但是连续 9 次「小」，就已经让这个策略无法继续进行。  

过了这个节点，能承受的次数加 1 。  

根据之前的拟合方程，大概需要 (102300 – 51200) / 46.2 = 1106.06 次赌博  


然后，我们再算一下，平均需要掷多少次骰子，才会出现一个连续 9 个「小」？  
## 3.1 期望  
让我们假设每一回合开始之前，都会有一个新的玩家加入游戏，与仍然在场的玩家们一同赌博。每个玩家最初都只有 1 元钱，并且他们的策略也都是相同的：每回都把当前身上的所有钱都押在正面上。运气好的话，从加入游戏开始，庄家抛掷出来的硬币一直是正面，这个玩家就会一直赢钱；如果连续 n 次硬币都是正面朝上，他将会赢得 2^n 元钱。这个 2^n 就是赌场老板的心理承受极限——一旦有人赢到了 2^n 元钱，赌场老板便会下令停止游戏，关闭赌场。让我们来看看，在这场游戏中存在哪些有趣的结论。  
首先，连续n次正面朝上的概率虽然很小，但确实是有可能发生的，因此总有一个时候赌场将被关闭。赌场关闭之时，唯n赚到钱的人就是赌场关闭前最后进来的那 n 个人。每个人都只花费了 1 元钱，但他们却赢得了不同数量的钱。其中，最后进来的人赢回了 2 元，倒数第二进来的人赢回了 4 元，倒数第 n 进来的人则赢得了  2^n  元（他就是赌场关闭的原因），他们一共赚取了 2 + 4 + 8 + … + 2^n = 2^(n+1) - 2 元。其余所有人初始时的 1 元钱都打了水漂，因为没有人挺过了倒数第 n + 1 轮游戏。  
另外，由于这个游戏是一个完全公平的游戏，因此赌场的盈亏应该是平衡的。换句话说，有多少钱流出了赌场，就该有多少的钱流进赌场。既然赌场的钱最终被赢走了 2^(n+1) - 2 元，因此赌场的期望收入也就是 2^(n+1) - 2 元。而赌场收入的唯一来源是每人 1 元的初始赌金，这就表明游戏者的期望数量是 2^(n+1) - 2 个。换句话说，游戏平均进行了 2^(n+1) - 2 次。再换句话说，平均抛掷 2^(n+1) - 2 次硬币才会出现 n 连正的情况。  

所以，出现 n 连「小」的时候，平均掷骰子的次数的期望为： 2^(n + 1) – 2 次。  
连续掷 9 次都是「小」，平均需要掷 2^(9 + 1) – 2 = 1022 次。  

而根据之前的数据，资金能成长到可以承受连续 10 次「小」，需要约 1106.06 次。  
也就是说，在黎明到来之前，通常已经出现了生命不可承受之重。  

即使你熬过了这个阶段，也无需骄傲，从 10 ~ 11 次的成长需要更久的时间，也带来了足够的几率将你打到。  

# 4 结论
此种策略，之所以看上去赢面非常大，实际上是因为一个非常容易忽略的原因，就是：  
**在这种条件下，一旦输，就是全部输光**。  

换言之，这个游戏从「赢，得 100 ；输，失 100」，变为了「赢，得 100 ；输，失 50000」。  

赢面当然会变大！  

再说白一点,Martingale Arbitrage **收益固定,风险几何级数增长**  

最后啰嗦一句：  
**珍惜生命，远离赌博**。  
