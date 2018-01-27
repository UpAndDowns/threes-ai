

<p align='center'>
<img src='./threes!/image/header.png'>
</p>

<p align='center'>
🤖 AI for the Threes! game. 🎮
</p>

# 灵感来源

几个月前和另外二位小伙伴一起参加了一个 AI 的比赛。虽然比赛结果不理想，至少我享受到了编程过程中的乐趣。从这次比赛中让我认识到 Go 除了写服务端，写游戏模拟器， AI 都是拿手好戏。最近微信跳一跳的辅助，冲顶大会的辅助基本也都是 Go 写的。于是我更坐不住了，也写一个来纪念我们那次比赛。


由于本人也是客户端出身，所以这个 AI 必须也能在手机上刷分。所以要找一个手机游戏，三个人可以玩的，或者名字带“三”字的，由此：


> Threes preson join one AI competition  ---> Threes-AI





# 游戏分析

Threes 的难点在于，这是一个**必输**的游戏。当游戏到了后半段，合成出了 6144 之后，很大一部分时间这个砖块所在的位置就不能动了，相当于 4 * 4 = 16 个格子，减去一个给它。场地上的砖块到后期也无法一下子合并，所以预留的空间很少，常常因为周转不开或者连续来 1 或者连续来 2，无法合成 3 ，活活被“挤死”了。

网页版设计过程中，并没有向客户端那样考虑“跳级”的模式，即出现的砖块能出现 3 的倍数的，比如出现 3，6，12，24，96，192，384…… 这些大额的砖块。所以网页版的游戏过程可能会比客户端上的简单一点。

为何说会简单一点？因为虽然不会出大的砖块（大砖块分值高），玩的时间会比较长，但是这样存活率也稍微高一点。如果连续的来，96，384，768，都来单个的，这样会导致棋盘上一下子出来很多不能合成的砖块，虽然分数会一下子暴涨，但是也会因为无法被合并，导致无法移动，迅速结束游戏。

在客户端上就存在“跳级”的设定，就可能一段时间就会出现这些砖块。我在测试 AI 的时候也发现了这个问题，被连续来单个的 1 或者连续的来单个的 2 逼死的几率不大，倒是被高分大砖块逼死的情况很多，这样导致存活时间不长，分数也没有网页版的高。


# Expectimax Search Trees 最大期望搜索树

在日常生活中，有些情况下，我们经过深思熟虑也不能判断这个抉择会导致什么样的结果？是好还是坏？

- 1. 摸扑克牌的时候，永远不知道下一张摸到的会是什么牌，拿到这张未知的牌会对牌局产生什么样的影响？
- 2. 扫雷游戏，每次随时点击一个方格，有可能是雷，也有可能是数字，雷的随机位置会直接影响该回合是否直接游戏结束。
- 3. 吃豆人游戏，幽灵的位置随机出现，直接导致接下来的路线规划。
- 4. Threes！游戏中，1 和 2 的方块随机出现，将会影响我们该怎么移动方块。

以上这些情况都可以用 Expectimax Search Trees 最大期望搜索树去解决这个问题。这类问题都是想求一个最大值（分数）。主要思想如下：

最大值节点和 minimax search 极大极小值搜索一样，作为整棵树的根节点。中间插入“机会”节点 Chance nodes，和最小节点一样，但是要除去结果不确定的节点。最后利用加权平均的方式求出最大期望即最终结果。

这类问题也可以被归结为 Markov Decision Processes 马尔科夫决策过程

## Expectimax 最大期望值的一些特性

其他的节点也并非是敌对的节点，它们也不受我们控制。原因就是因为它们的未知性。我们并不知道这些节点会导致发生什么。

每个状态也同样具有最大期望值。也不能一味的选择最大期望值 expectimax，因为它不是 100% 安全的，它有可能导致我们整个树“松动”。

机会节点是由加权平均值概率管理的，而不是一味的选择最小值。


## 举例

举个例子：

玩家需要以得到最高分数为目标去行动。假设每一步的概率都相同。

```
伪代码


```


## 关于剪枝

**在 expectimax 中不存在剪枝的概念**。

首先，对手是不存在“最佳游戏”的概念，因为对手的行为是随机的，也是未知的。所以不管目前期望值是多少，未来随机出现的情况都可能把当前的情况推翻。也由于这个原因，寻找 expectimax 是缓慢的（不过有加速的策略）。




## 效用函数 Utilities

在 minimax search 极大极小值搜索中，效用函数 Utilities 的缩放比例并不重要。我们只是希望能获取到更好的状态以致于获得更高的分数。（这种行为也被称为单调变换不敏感性）


在 Expectimax Search 期望最大值搜索中，效用函数 Utilities 一定要给定一个有意义的值能比较出大小，因为这里对大小极为敏感。


## 概率函数

在 Expectimax Search 期望最大值搜索中，我们有一个在任何状态下对手行为的概率模型。这个模型可以是简单的均匀分布（例如掷骰子），模型也可能是复杂的，需要经过大量计算才能得到一个概率。


最不确定的因素就是对手的行为或者随机的环境变换。假设针对这些状态，我们都能有一个“神奇的”函数能产生对应的概率。概率会影响到最终的期望值。函数的期望值是其平均值，由输入的概率加权分布。

举个例子：计算去机场的时间。行李的重量会影响到行车时间。

```
L（无）= 20，L（轻）= 30，L（重）= 60

```

三种情况下，概率分布为：

```

P（T）= {none：0.25，light：0.5，heavy：0.25}


```

那么预计驾车时间记为 

```

E [L（T）] =  L（无）* P（无）+ L（轻）* P（轻）+ L（重）* P (重)
E [L（T）] =  20 * 0.25）+（30 * 0.5）+（60 * 0.25）= 35


```



## 数学理论

在概率论和统计学中，数学期望(mean)（或均值，亦简称期望）是试验中每次可能结果的概率乘以其结果的总和，是最基本的数学特征之一。它反映随机变量平均取值的大小。

需要注意的是，期望值并不一定等同于常识中的“期望”——“期望值”也许与每一个结果都不相等。期望值是该变量输出值的平均数。期望值并不一定包含于变量的输出值集合里。

大数定律规定，随着重复次数接近无穷大，数值的算术平均值几乎肯定地收敛于期望值。


https://web.uvic.ca/~maryam/AISpring94/Slides/06_ExpectimaxSearch.pdf




# Minimax search 极小极大值搜索


https://github.com/rianhunter/threes-solver

# Monte Carlo tree search 蒙特卡洛树搜索


https://www.zhihu.com/question/39916945

https://en.wikipedia.org/wiki/Monte_Carlo_tree_search#Pure_Monte_Carlo_game_search

https://www.jianshu.com/p/d011baff6b64


https://juejin.im/post/59f16e8c5188250385371302


http://codingpy.com/article/monte-carlo-tree-search-and-alphago/
