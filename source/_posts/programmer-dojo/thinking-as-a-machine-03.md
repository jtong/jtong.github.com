---
title: 像机器一样思考（三）—— 穷尽就是力量
date: 2016-08-25 22:14
---

经过前两篇的内容学习，我相信大家已经差不多学会了这个思考模型。本篇的重点是用它来解决更复杂的问题。当我们开始解决一些稍微复杂点的问题的时候，我们会发现差不多的态度是不行的，我们需要严谨的态度进行缜密的思考才能真正发挥出这个思考模型的力量。

慢慢你会发现，这个思考模型本身不会让你思维缜密，而思维缜密了才能用好这个思考模型。它带来的最大的好处，是让你自己开始看到自己思维的欠缺，从而不再是个思维世界的盲人。

## 穷尽

那么，我们开始做点复杂的题目。也没复杂多少，我们扩展一下上一篇的题目，算学生的成绩单：


打印所有人的成绩单。已知输入的格式是
> ["学号", "学号", "学号"]
> 比如： 
["TWA20160101",  "TWA20160102", "TWA20160103"]

我们有一个全局函数可以给我们提供所有的学生的成绩：

```
function loadAllScore(){
    return [{
        name: "张三",
        id: "TWA20160101",
        chinese: "95",
        english: "80",
        math: "95",
        programming: "80"
    },
    ....
    ]
}
```


要求打印出成绩单类似于：

```
成绩单
姓名|数学|语文|英语|编程|平均分|总分 
========================
张三|75|95|80|80|82.5|330
李四|85|80|70|90|81.25|325
========================
全班总分平均数：xxx
全班总分中位数：xxx
```

仅对这个题目进行划分，我们一定觉得很简单，对吧。就仿照着之前的写呗：

```
#1 生成成绩单view model
输入：
  studentIds: [String]
输出：
  scoreSheet: {
    studentScores:[{
        name: String,
        chinese: String,
        english: String,
        math: String,
        programming: String,
        average: String,
        summary: String
    }]
    summary: {
      totalAverage: Number,
      totalMidden: Number
    }
  }
#2 打印成绩单
输入：
  scoreSheet
输出：
  result: String
```

我相信很多人都是这么想的，然而不幸的告诉这么想的同学，这种写法是错误的。原因其实也很简单，请问，#1输出里的chinese等成绩是怎么得到的呢？没有来源吧？你说你调了```loadAllScore```函数？那为什么不写在输入里呢？

所以说，我们遗漏了一些输入。回到我们开始的标题上：穷尽。

可能有很多人听说过一个分析问题的基本原则：完全穷尽，各自独立。很多人听到这个时候，会很困惑：穷尽什么？独立什么？经过我们这些练习，我相信在编程领域，你们这个困惑会小很多。


所谓各自独立，说的就是在我们划分任务的过程中，每一个任务都对应一个代码块或一个函数，这些代码块和函数，是互相不包含的（不是不依赖，这是翻译的问题，各自独立的独立指的是Exclusive不是Independent）。

所谓的完全穷尽，说的是我们需要穷尽这个代码块或函数里所有的输入和输出。不能遗漏任何一个输入，任何一个输出。我们的每一项，它的属性，也不能有遗漏，我不能说分析```studentScores```只想到部分属性，比如说：

```
studentScores:[{
        chinese: String,
        english: String,
        summary: String
    }]
```

这样是不行的。如果我们不严于律己穷尽所有的数据项，我们就会在写代码的时候遇到各种问题。能否穷尽与否，也看出来你思维的缜密与否。

是不是开始感觉到麻烦了，刚开始做的时候是有些慢的，但我们坚持穷尽这个好习惯，就会渐渐的感受到自己能力的成长。如果你穷尽了所有的输入输出，那么各种可能遇到的问题就像是如来佛手里的孙猴子，无论有什么变数也尽在你掌握之中了。

那么如果我们穷尽输入输出的话，我们这个题目真正的任务应该怎么分解呢？可以写成这样：

```
#1 获得学生成绩
输入：
    studentIds: [String]
    studentInfo: [{
        id: String,
        name: String,
        chinese: String,
        english: String,
        math: String,
        programming: String,
    }]
输出：
    studentScores:[{
        name: String
        chinese: String,
        english: String,
        math: String,
        programming: String,
        average: String,
        summary: String
    }]

#2 计算总计
输入：
    studentScores
输出：
    summary: {
      totalAverage: Number,
      totalMidden: Number
    }
    
#3 打印成绩单
输入：
    studentScores
    summary
输出：
    result: String
```

可能你会奇怪，为什么会分成3步呢？或者说，我们该怎么判断分几步呢？这其实没有一个标准答案，我建议初学者尽量步子小一点，多分几步，经验丰富的人就可以步子大一点。不过有一个反直觉的经验可以分享给大家，你步子大了，开发速度不见得快，因为人是会犯错的。

这个题是写出来了，但是我们还是不太清楚怎么穷尽对吧。说是穷尽输入输出，到底输入输出都有多少大类呢？这个也是可以穷尽的。

输入总共有下面几大类：
1. 参数
2. 读取全局变量
3. 调用全局函数后得到的返回值
4. 读取局部作用域变量（比如this）
5. 调用局部函数后得到的返回值
6. hard code的数据

输出总共有下面几大类：
1. 返回值
2. 修改全局变量
3. 调用全局函数时传的参数
4. 修改局部作用域变量（比如this）
5. 调用局部函数时传的参数

## 来去

听起来不错，不过从哪来，到哪去，还是要写清楚的，我们的studentInfo从哪里来？我们的result又到哪里去了？加上这个来去，我们最终的版本是长这个样子的：

```
#1 获得学生成绩
输入：
    studentIds: [String]
    studentInfo: [{
        id: String,
        name: String,
        chinese: String,
        english: String,
        math: String,
        programming: String,
    }]: loadAllScore()
输出：
    studentScores:[{
        name: String,
        chinese: String,
        english: String,
        math: String,
        programming: String,
        average: String,
        summary: String
    }]

#2 计算总计
输入：
    studentScores
输出：
    summary: {
      totalAverage: Number,
      totalMidden: Number
    }
    
#3 打印成绩单
输入：
    studentScores
    summary
输出：
    result: String: console.log()
```


## 练习

引入加分策略
少数民族 +10分
体育特长 +20分
艺术特长 +15分

# 题外话

好像输入和输出的可能性太多了，这很容易让人乱啊。
是这样的，所以为什么到了函数式编程我们需要强调纯函数，只有一个输入来源和一个输出去处，一般来讲就是输入只有参数，输出只有返回值。所以你看，如果你把一个领域都穷尽掉，你也会自己发明出那些靠谱的实践，换句话说，如果你能在用这套思维模型的过程中逐渐发现跟各种最佳实践都很容易配合使用，那就对了。（其实也不是什么配合使用，因为它跟代码是等价的，所以容易写代码的方法就容易用它，这是当然的。）