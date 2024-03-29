---
title: 许愿驱动开发（一）：导入介绍
date: 2024-01-26 00:49
---

## 起因

2023年LLM火了之后，有一些研发组织可以用chatgpt，所以他们试图用chatgpt来提升开发效率，当然主要是编程提效。我有幸帮助这样的组织进行了一系列的培训和咨询。

开始的时候，我按照我前面《ChatGPT编程秀》里面的内容教开发人员老老实实写提示词，然而在过程中遇到很多阻力。因为正如很多老读者了解的，要想哪怕完成编程这一步，也要完成四步：

- 先根据需求生成出输入输出的关键要素，包括函数签名还有外部依赖的接口都要定义出来。
- 然后根据定义出的输入输出关键要素和需求，生成测试的文本用例。
- 然后根据输入输出、需求和文本用例，生成测试数据，有时候直接就是测试代码。
- 最后才根据输入输出、测试数据、需求，生成代码。
  
而且我们在生成代码的时候发现稍微复杂点的需求，哪怕你给足测试数据，他也不能一次生成成功。因为大模型生成代码的能力有限，所以大模型编码也需要一步步走，需要先给一组测试数据，生成一个简单的代码。然后把已经生成的代码和下一组测试数据放入提示词再生成新的代码，直到得到可以实现全部测试用例的代码。整个过程可能如下所示：

![提示词驱动开发的简化流程.png](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/01-introduce/01-prompt-driven-development-process.png)

其实整个过程就是TDD的流程（很神奇的是TDD的主要步骤在用LLM写代码的过程中竟然一步都不能省，说明这个方法还是很科学的。）写到这里，我相信很多在公司里安利过TDD的人就能明白我前面说的遇到很多阻力是什么了。TDD这个步骤太麻烦了，虽然以前是人来完成流程中的每一步，现在人只需要扮演一个流程的主持人，每一步实际上是LLM完成的。但是在实操过程中，依然听到很多人觉得很麻烦。

首先因为人扮演主持人也不是没有工作量的，其中最麻烦的是要构造提示词，这个提示词要经常从工程文件夹里四处找，然后拷进来，还要转化为提示词，还要把之前生成的提示词存下来。更不要说这四步要四种思维方式，人是不喜欢切换上下文的，如果没有经过刻意的训练，并不会觉得这四步好做，人的惰性很难接受。

经历过几轮之后，我对于教会大多数人用TDD的方式写提示词来驱动LLM写代码已经放弃了。我的人生经历告诉我，不要跟人的惰性对抗，还是顺着他们好了。所以我不禁思考起来，能不能顺着人们偏好的方式来？

## 思考

那么人的偏好是什么呢？我在使用的过程中发现，没有被培训过的程序员大都像许愿一样的写提示词：
- 比如有个客户试验的场景是I/O合并，程序员就会直接要求LLM对I/O按照地址进行排序，然后把地址相连的I/O合并起来。这个就很有问题，因为I/O在代码里是由什么数据结构表示的呢？地址是哪个属性呢？怎么算相连呢？合并了之后怎么下发呢？这些概念，不同的代码实现，表示的含义也完全不一样。自然LLM也无法给出你可以直接用的代码，他只能自己构造一些数据结构和外部依赖来示例。
- 无独有偶，另一个客户也有一个类似的案例，他们要做股票拆单，给出的都是抽象的股票交易术语，完全没有给出订单的定义（母单没有定义、子单也没有定义）、订单从哪来，到哪去，更不要说拆单的逻辑了，那术语更专业。LLM也表现出了与上一个案例中一样的行为表现，自己构造了数据结构和外部依赖。

这种许愿行为几乎在每次新学员学用提示词写代码时必然会发生的，已经是绝大多数程序员的普遍行为。我们分析一下，为什么绝大多数的程序员觉得这样说就够了，而LLM却因为信息不够写不出令他满意的代码，这其实背后隐藏着一个人说话的习惯问题：

- 每个人说话的时候，是会基于自己认可的一个背景上下文，说出自认为没有歧义的话。在这种自认为的情况下，人们还总会说省力的话，也就是最少的话表达最多的意思。
- 在这种习惯下，人是不爱在说话的过程中时刻考虑对方的背景上下文和自己的背景上下文的交集，并尽量追求的无歧义的，这个太累了。人喜欢跟自己背景上下文差不多的人交流，尤其是在交代任务的时候。

所以如果这个程序员把LLM当成他项目的团队成员，那么他说话没有任何问题，因为他项目里的团队成员是跟他有一样的上下文的，从完成编程任务的角度来看，跟他是一样的。所以他说的话他们也能理解，也能写出符合期望的代码。然而LLM并不是他项目里的同伴，它是无状态的，根本不知道你项目的上下文，跟他是不一样的。

于是我们从外面看过去，就会觉得程序员总是表现出一种许愿的行为模式。既然这样，那我们就应该想办法，把程序员许的愿望的上下文给补齐，让程序员说的话在这个上下文下是没有歧义的，那么理论上LLM不就跟这个程序员一样了么？然后把上下文加上程序员许的愿望一块扔给LLM，不就能写出符合这个程序员预期的代码了么？

![this-one-add-this-one](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/01-introduce/02-this-one-add-this-one.png)

## 思路

于是接下来我就考虑了一下，程序员写代码的时候主要有哪些上下文呢？
- 我一般接到一个需求的时候，都是先从ide的文件里找相关文件。所以首先我们需要一个项目文件树。
- 然后是打开相关文件后，读我要改的代码、我依赖的代码、资源文件、建表语句等等。所以我们需要这些相关文件的内容。

只要上面的上下文内容提供了，任务就变的没有歧义了。整体上就是这样的东西：

![许愿驱动开发的提示词模版和示例](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/01-introduce/03-wish-prompt-template.003.png)

（可以看出来，我们连运行时都可以用dockerfile的形式表达出来。感谢这么多年来开源社区一直的努力，软件领域里面几乎没有什么知识是不能文本化的。）

这里面只有任务描述是人工编写的，而上面的内容完全可以靠机器生成，最多人工调整一下范围，读取并生成完全可以自动化。

然后我们只需要把这个提示词贴到大模型里就可以得到代码了。有的时候我们会担心写的任务太抽象了会发生什么？你会发现LLM会直接给你分解任务，而不给你代码。起码ChatGPT可以做到。然后你再把子任务补齐上下文就可以得到代码了，基本流程如下：
![wish-based-task-breakdown](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/01-introduce/04-wish-based-task-breakdown.png)

经过测试，最好是只分解一次任务，因为超过一次分解，如果出现了幻觉，需要人来判断前两级任务的正确性，这对于人的能力要求更高。比较起来，要求人不要写出超过一次分解的任务描述，这个更容易一些。

有了这个东西，写程序的过程就真的像是许愿了，给大家看看我许过的愿望：

```
我想要设计用户角色，用于管理其他为服务的权限问题，给我一个方案

我想实现能给用户assign role的rest api

我希望给RoleController编写测试，基于Spring MockMVC

我希望执行 fileExplorer view 的bar上有一个按钮，点击可以刷新fileExplorer view。

我希望 点击 file explorer 的 refresh 可以重新加载config文件，然后刷新整个file explorer

我希望 服务端可以返回一个表单，客户端可以渲染这个表单。我希望fetch /replay 在某些情况下返回，目前你可以设计成如果message含有某字符串，返回这个表单。
```

可以看到从前端到后端，从设计到实现，它都可以帮我们，而且因为给足了上下文，所以我也不用给他多解释什么，他就可以实现我想要的代码。

基于这个思路，我做了一些小工具来完成生成上下文的这个效果，下一篇开始慢慢教大家怎么用这个工具达成许愿驱动开发的效果。

那么在实践的过程我也发现，这个思路对于LLM也提出了最关键的需求：上下文窗口要大。基本上这样贴进去的上下文，垒出个小1000行代码真的是轻轻松松。这还是我自己做一些实验，如果正式的项目，我觉得几千行代码也是很正常的。如果上下文窗口太小，那么恐怕就很困难了。所以国内训练LLM的同学们，也请加油，上下文窗口越大越好，谢谢。
