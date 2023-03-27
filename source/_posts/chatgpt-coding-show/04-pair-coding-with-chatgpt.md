
## 以终为始

按照上一篇的架构，我们整个程序最后写完的运行视图大概是下面这个样子的：
![final-process-view](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/chatgpt-coding-show/04-pair-with-chatgpt/04-01-final-process-view.png)

可以看出我们的实际结果比我们上一篇文章考虑的还要复杂：

- 因为AI生成的API不接受批量prompt处理，所以我们需要展开负责表达简单排列组合的DSL-b生成一批DSL-b'，再由DSL-b'一对一转成DSL-c
- 因为API的限制，不是服务端生成文件让我们去下载而，而是直接把base64给我们发回来，所以不得不自己存文件还得考虑太多了怎么分分文件夹
- 因为我们的目标是了解哪些关键字和其图片好用，还得写点信息到png里，不然光看个图片能看出什么来？(其实AI也写了一些进去，但是毕竟是混编后的，不太好读)。而由于nodejs技术欠缺，不得不还要调用python

但总的来说，它还是上文那个架构，只是每一层都只写了一个节点，毕竟个人用，也不需要那么多扩展。而即便如此也是个挺复杂的结构，这么复杂的结构，自然不能一下生成出来，我们需要分块实现，那么我们的每一块要怎么实现呢？有什么套路呢？这将是我们接下来的内容，内容比较长，这一篇我们先讲生成代码这一步的套路。

## 如何编写一个节点

### 编写主干逻辑
我们以dsl_translator这个节点为例，一开始，我只是这么提问（并没有，我只是演示一个错误示范）：
> 我想用nodeJS用下面的yaml描绘的数据结构得到一个新的数组:
		    
``` yaml
base:
  steps: 10
  batch_size: 1
poly:
  - template_prompt:
      template: >
          a cat,
          ${ chara }
          ${ facial_expressions }
      meta:
        view_angle: 
          - side_view,
          - front_view,
        faal_expressions:
          - (smile:1.5),  
          - (smile:1.2),  
          - smile,  
```

果然，ChatGPT也不惯着我，给出了我下面的代码：


``` js
//......
const newArray = [];
data.poly.forEach(item => {
const obj = {
    template: item.template_prompt.template,
    meta: item.template_prompt.meta
};
newArray.push(obj);
});

console.log(newArray);
```

我仿佛听到AI在说：“你就说是不是数组吧，你要数组，我给你数组，没毛病~”

遇到这种问题，只能反思是自己没说清楚，那么该怎么说清楚呢？

我们这个程序是服务于AI画图的，其实我们也可以从AI画图中借鉴不少经验，比如说在AI画图中，有一个control net里的open pose技术，我可以画个骨架，就像下面这样：
 ![open-pose-example](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/chatgpt-coding-show/04-pair-with-chatgpt/04-02-open-pose-example.png)

我就可以用这个骨架图去画个穿靴子的猫，拿着个刺剑什么的。这个技术告诉我们什么呢？我们可以通过描述一个骨架来控制AI生成的内容。那么图片可以，编程是不是也可以？

于是经过一段时间的摸索，最后我写出了下面的描述：

>我想用nodeJS用下面的yaml描绘的数据结构得到一个新的数组:

```yaml
base:
  steps: 10
  batch_size: 1
poly:
  - template_prompt:
      template: >
          a cat,
          ${ chara }
          ${ facial_expressions }
      meta:
        chara: 
          - Abyssinian,
          - cat_in_boots,
        facial_expressions:
          - (smile:1.5),  
          - (smile:1.2),  
          - smile,  
```
		    
>要求：  
>1. 假设上面的yaml转成json的转换代码我已经写完了
>2. 我需要遍历poly下的所有的顶层元素
>3. 遍历过程中，要处理template_prompt元素的子元素：
>    1. 从template中读取作为模版。
>    2. 读取meta中的属性，因为属性可能每次都不一样，是不确定的，所以不能硬编码。
>    3. 然后基于meta中的属性，把template作为 string literal 解析，这个解析代码我已经有了，假设名为render_string_template，可以不实现，留一个函数接口即可。
>    4. 要遍历组合meta中的属性形成一个数组，比如chara 有两个值，facial_expressions 有三个值，那么应该生成2*3也就是六组属性放入这个数组中，这个数组和template会被传入render_string_template函数，最后会获得两个prompt字符串
>4. 将生成的prompt字符串数组和template_prompt元素之外的其他元素合并成一个对象，要求在同一级别。prompt字符串数组有几个元素，就会合并成几个对象，并放入一个新数组中，我们称之为ploys。
>5. 继续遍历，直到遍历完所有顶层元素，所有元素都放入了polys中。polys是一个一维数组。
>6. 将ploys中的每一个元素与base中的属性合成一个新的对象，base的属性展开与prompt属性同级，当ploys中的每一个元素的属性名与base中的属性名相同时，覆盖base中的属性。这些新对象组合出的数组就是我要的数组

可以看到，我在之前的prompt的后面加了很多的描述，其中的2-6就是用自然语言讲了我期望这个函数实现整个过程的大概逻辑。这个手法就很像Open Pose。通过这个操作，我得到了我想要的代码，毕竟这个逻辑不复杂。

### 边界划分

上面prompt的要求里除了表达了主干逻辑，其实也用了一些其他的技巧。比如“1. 假设上面的yaml转成json的转换代码我已经写完了”，“把template作为 string literal 解析，这个解析代码我已经有了，假设名为render_string_template，可以不实现，留一个函数接口即可。”

这个呢有点像control net里的seg技术，他就是把图片上的东西都分区域，就像下面这样：
![seg-example](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/chatgpt-coding-show/04-pair-with-chatgpt/04-03-seg-example.png)

然后再生成的就靠谱很多，甚至你还可以对其中特定部分进行针对性的描述。这个技术就很适合用来控制AI生成的注意力焦点。

那么在我们这个例子，上面提到的那些描述就起到了seg的作用。他可以让我们不用在一个prompt描述里编写所有的细节，这对于复杂的逻辑很有帮助。因为即便是一个小节点，其实逻辑也可能有点复杂的，就像这里面，我们可以把meta数据与tempalte合并的功能延迟实现，只描述与他的交互，使生成的程序先把参数扔给它。有的时候，我们想要使用特定的库，也可以用这个方式，比如我在另外一个场景下是这么实现的这个效果：

```prompt
1. 读取文件的fs，要使用const fs = require('fs/promise')引入。
2. 用js-yaml库解析yaml。
```

通过这样的技巧，我们就可以把代码进行进一步的分解，让我们更容易描述清楚我们想要部分的业务逻辑，而且也可以给我们节省点算力，毕竟ChatGPT生成的时候如果太长也会中断，你让他继续的话，有时候也续不上。这个技巧就可以让他专注于你希望他专注的地方，从而提高表现力。

### 功能迭代

上面的写完之后呢，我发现一个问题，这个DSL还不是我想要的最终形态，对于同一套模板，我可能需要多种meta，因为有些属性的组合是没有意义的，我也不想浪费我宝贵的GPU。那么我们就不能太暴力的让它穷举所有的组合，我要针对同一个tempalte给他不同的属性组合，所以meta的值就必须是个列表，大概如下所示：

``` yaml
base:
  steps: 10
  batch_size: 1
poly:
  - template_prompt:
      template: >
          a cat,
          ${ chara }
          ${ facial_expressions }
      meta:
        - chara: #  这里改成了数组
            - Abyssinian,
            - cat_in_boots,
          facial_expressions:
            - (smile:1.5),  
            - (smile:1.2),  
            - smile,  
```
于是现在我们就面临一个问题：改代码。而这个时候就体现出我们之前拆任务的价值了，因为各个模块都是隔离的，那就没有什么改代码，重新生成一份就好了，所以我改了改要求：

> 要求：
>	1. 假设上面的yaml转成json的转换代码我已经写完了
>	2. 我需要遍历poly下的所有的顶层元素
>	3. 遍历过程中，要处理template_prompt元素的子元素：
>		1. 从template中读取作为模版。
>		2. 读取meta中的属性，因为属性可能每次都不一样，是不确定的，所以不能硬编码。
>		3. 然后基于meta中的属性，把template作为 string literal 解析，这个解析代码我已经有了，假设名为render_string_template，可以不实现，留一个函数接口即可。
>		4. 要遍历组合meta中的每一个属性组形成一个数组，
>			1. 每一个属性组可能只需要看做一个对象，当且仅当每一个属性值都为单值
>			2. 每一个属性组可能也需要展开，当且仅当任何一个属性值有多值，比如 chara 有两个值，facial_expressions 有三个值，那么应该生成2*3也就是六组属性放入这个数组中，这个数组和template会被传入render_string_template函数，最后会获得两个prompt字符串
>	4. 将生成的prompt字符串数组和template_prompt元素之外的其他元素合并成一个对象，要求在同一级别。prompt字符串数组有几个元素，就会合并成几个对象，并放入一个新数组中，我们称之为ploys。
>	5. 继续遍历，直到遍历完所有顶层元素，所有元素都放入了polys中。polys是一个一维数组。
>	6. 将ploys中的每一个元素与base中的属性合成一个新的对象，base的属性展开与prompt属性同级，当ploys中的每一个元素的属性名与base中的属性名相同时，覆盖base中的属性。这些新对象组合出的数组就是我要的数组

改完要求，配上上面的DSL直接扔给了它，得到了我们想要的代码。

所以在今天这个时刻，有些重构工作突然变得没有那么大价值了，因为在划定好的边界里，重写比重构更快，尽管两次输出的代码并不一样，但是他们的功能是一样的。

## 总结一下
我们首先概述了整个程序的最终运行视图，以及在实现过程中遇到的挑战。然后我们以dsl_translator节点为例，讲解了一些与ChatGPT结对编程的一些套路：

首先是编写主干逻辑，通过描述我们期望函数实现的逻辑，类似于Open Pose中的骨架图，以控制AI生成的内容。我发现很多人让ChatGPT工作的时候总想给一句话需求，软件还是很复杂的，必要的话，我们还是要把主干逻辑写出来。程序员有多讨厌一句话需求，我想我不用多说了，到咱们自己提需求的时候，可不能忘本。

接着我们介绍了通过类似于Control Net中的seg技术，我们将代码进行更细致的分解，这样的技巧可以使AI专注于我们希望专注的地方，同时让我们自己也更容易描述我们想要的业务逻辑，既节省算力，也能提升AI的表现。毕竟GPT-4还没有广泛使用，3000字还是个门槛。其实即便是广泛使用了，我们不要忘了，AI这个东西生成总是不太稳定的，如果你切得足够小，成功率总是要高得多，而且你可以靠反复生成来达成某种自动化，这个呢，我们下一篇文章讨论。

接着我们面对了一个需要改代码以加入新功能的场景，因为我们之前已经进行了设计，既然已经拆分好了模块，重新生成一份代码来实现新的功能，比改代码更快。第一次使用这个技巧的时候，我想起了流量地球里550C接入的时候，发出的提示音：正在生成底层操作系统。是的，对于AI来说，重写比重构更快。这个点极大地动摇了软件工程中好多具体实践的根基，从这一刻起，面面俱到的文档胜过了可以工作的代码。具体到行为上的变化就是，我现在写代码都会有意识的保存自己对AI提问的prompt，并且我会在每个节点的根路径上记录下完善的技术文档。

到此为止，我们把与ChatGPT结对写代码中，怎么第一次把代码写出来这一步的主要技巧给讲的差不多了。但其实这还远远不够，正如没有ChatGPT的时代我们就知道的，第一次把代码写出来所占软件开发的成本少得可怜，软件开发成本的绝大部分都是在后续迭代中花掉的。那么对于一个节点也是一样的，尽管我们展示了一种重写优于重构的新思路，但这一篇也仅仅讲了个思路，AI生成的不稳定性依然是悬在我们心头上的一把刀，所以我们接下来会讲讲更多的工程实践来确保我们持续写出可信的代码。
