---
title: 架构可视化的坏味道
date: 2018-07-07 15:51
---


## 抽象的坏味道

[上文](https://jtong.github.io/2020/01/30/something-about-software-development/visualize-arch-design-introduce-c4/)说过，C4说穿了就是几个东西：关系-线、元素-方块和角色（角色不过是图形不同的方块）、关系表述-线上的文字、元素的描述-方块里的文字，虚线框（如前文所说，在C4里面虚线框的表达力被极大的限制了。）

这些东西一点都不新，我们自己随便找个白板，无非也是用这几个东西来表达架构，它的优点在于引进了一些分层，使得我们思路不是特别混乱，容易给别人看懂我们的思路，也容易帮助自己整理思路。

所以C4不能帮你做好架构设计，但是它能让你的设计中的问题暴露出来。被自己或其他人纠正。

可视化的威力就在这里，但根据我的经验，即便你用上了C4也不见得就能表达清楚，不过好消息是，终于我们可以聊一些高级的表达问题了。

可视化之后，我们能看到自己的表达问题，大概的问题有两个：抽象层次和抽象粒度。这个是表达方面永恒的问题，也就是软件设计永恒的问题，没有万灵丹，但是用上了可视化手段之后还是有机会让生活更美好一点的。


这两个问题可能太抽象了，不容易意识到，那我们可以看图，从图上的具体表现来发现坏味道。一般会有几个迹象表明我们有可视化的坏味道：
1. 一张图上过分密密麻麻的线
2. 一张图上太过多元素（也就是方块）也是坏味道
3. 一张图上太少的元素，比如角色特别少
4. 每个图上文字表达不契合，有的太泛泛，有的太细节也是问题。
5. 无限制的画更多张图，基本上也就失去了使用图形化表达的意义。


那么对应的手段就有：

## 合成更大的元素

当我们发现密密麻麻的线、太多的元素，闻到这个味道的时候。我们可以考虑是不是该把里面的一些元素合成更大的元素了。Component可以合成Container，Container可以合成System，这样就会分成更多的图，每张图就变得没那么多线和元素了。

紧接着会面临下一个问题：怎么合成一个更大的系统，Container是明确的，所以Component合成Container不是问题，问题是Container怎么合成一个系统，为什么是这些Container合成这个系统，而不是另外几个？或者多加几个、减几个？

这个问题没有标准答案，但是有一些其他的框架可以提供一些思考的维度。

比如可以结合akf扩展立方来思考



![akf扩展立方](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/bad-smell-of-visualize-arch-design/pic-01.png)



X轴就比较容易，一方面看你的容器本身的描述来发现设计上是不是支持横向复制的，另一方面则是看你的部署图。
Z轴相对难一些，只是比较偏技术。比如当技术上有性能瓶颈，则需要注意这一个维度，有时不得不搞出一些特殊的容器出来，有时已经存在这些容器了，他们可能单独属于一个系统（类似于大数据分析的系统），或者一个系统的某一个局部（这就是我说的虚线框的表达力被限制的地方）。

Y轴给人的感觉是最容易操作的，但实际上却是最难的做好的，Y轴的背后是业务，往往我们觉得就按业务切成多张图就好了么。这种想法就表现出我们其实很看轻理解业务的难度，于是也总是出问题的地方。如果你能跨过这个心理障碍，决定去认真做一下，那么也有一些工具可以帮助我们做好。



![领域模型与架构设计](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/bad-smell-of-visualize-arch-design/pic-02.jpg)


最经典的工具组合就是求助于DDD，结合康威定律和步速，考虑维护的团队、使用的角色、变化的节奏，这块展开就复杂了，有机会再聊。

这里说一个最简单的做法。按照用户角色分。同一种角色，由于它的，公司里的职能，他的职责都是已经被定好的。天然在系统上就有一种隔离性。比如招聘专员、会计、出纳。他们使用的系统肯定是不一样。

但说简单，其实也不简单。我见过一些图，上面的角色只有两个，内部用户和外部用户。而另一些图，细化到了persona的级别，或者把职级都放上去了。所以无论再简单的原则，最后都会掉进抽象的坑。


## 画一些共识图来忽略掉一些通用的元素

有时候合成了更大的元素，元素依然很多，线条依然很密。画多张图也不够切分的。这个时候我们可以求助于共识。

人与人交流，彼此之间如果已经有一些共识存在就可以少废很多话，共识多到一定程度只需要确认一个眼神就完成交流了。所以毫无疑问做好共识管理，就可以大幅简化我们的架构图。

所以在我们做架构可视化的时候，经常会先画一个技术共识图，比如以一个我们的能力建设的数字平台为例，我们就画了一个下面这样的技术共识图。：

![技术共识图](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/bad-smell-of-visualize-arch-design/pic-03.png)
 
然后在后面画具体的图的时候，我就可以省略掉一些共识的元素，像nginx和数据库就没有了，可以更关注在业务上，而不是技术上来画图。

## 通过制定主题，限制文字的抽象层次

其实上面的技术共识图就是类似的做法，只是用于技术方面，如果用于业务方面，我们可以用一些抽象的名词或动词来代替一类业务，比如下图：

![数字平台系统景观图](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/bad-smell-of-visualize-arch-design/pic-04.jpg)


上图是一个系统景观图。当前这个主题是希望，人们一眼看清楚这个系统里面的相关角色都在使用什么系统，并且他们关注什么，职责是什么。所以具体学什么，怎么学的，都不是那么重要。所以我们就用学习一词代表了一系列的业务。

当主题确定的时候，很多纷杂的信息就没有了。一定要克制住自己，试图在一张图上，表达足够多信息的冲动。


## 只画重要的图，剩下的交流的时候再画。
除了像上面说的，不要试图在一张图上给他足够的信息。同时也，不要试图把所有的信息都表达出来。

绝大多数的图可能只在交流具体业务的时候才画，推荐使用动态图。
这个手边没有例子，找到再说吧。

## 总结

即便有了C4这么，好用的可视化工具。我们依然会看到，自己会掉进抽象的坑。所以在使用的时候一定要注意坏味道，经常检察是不是犯了抽象层次和抽象力度的错，才能做好可视化。这件事上，没有谁能幸免，所以要时常自省，与诸君共勉。