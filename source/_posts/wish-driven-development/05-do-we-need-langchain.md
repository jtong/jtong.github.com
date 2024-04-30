
## 分析
上回说到我们准备落代码，要把这个逻辑串起来，我们可能需要一些框架，所以肯定很多人都会想到LangChain。

但不要着急，我们来考虑一下，我们为什么需要LangChain？我们真的需要LangChain吗？实际上，很多人都没有仔细思考这个问题，就是习惯的觉得做什么开发就先找一个框架。因为我们觉得框架可以提高开发的效率，这个习惯做得多了，就忘记了思考为什么框架可以提高我们的效率。

其实新的时代来临时，人们在一开始总是会抱着旧时代的思想钢印在新时代生活一段时间。比如在苹果诞生以前，我们的智能手机还要有一支笔，而且屏幕哪怕已经不小了，左下角还有一个开始菜单。所以当LLM的时代来临的时候，人们也会有这么一阵，而LangChain就是这个时代的产物。

在LLM诞生之前，我们提供工具的思路是封装一系列的功能，将一堆接口提供给用户，这对接口通常也会伴随着一系列的概念。我们的工具会要求用户把他们的需求的数据翻译成我们工具里的概念，从而能最大限度的减少用户的开发成本。因为对于人来说，实现功能相关对复杂一些，毕竟我们要做类似查资料找到对应的API、想实现思路、写代码、调试等等工作，思考成本高，也容易写错，而在理解了接口之后，准备接口所需的数据结构成本是偏低的。而进入LLM时代后，如果是简单的小功能，这些都是LLM写的，对于LLM来说写这些东西没什么成本，而翻译数据结构对LLM反而是难的，主要是不同的接口都有不同的子概念，所以这个时候，进行翻译概念、转换数据结构成本反而变高了。

而且LangChain还有个问题，它太新了，太新就导致了ChatGPT里面没有它的知识。那么可不可以把LangChain的文档喂给ChatGPT，让它学会使用LangChain呢？理论上没啥问题，问题是，LangChain的文档太大了，超出了上下文窗口的长度，而且别忘了，我们可是要做许愿驱动开发，那我们项目内的上下文它也不小，你忍心从每次提问的里面切掉多少给LangChain的知识呢？

所以LangChain这种搞法的库更多的还是适合在手工写代码的时候提高效率，而我们前面的库和插件都是LLM写的，怎么到了这个Agent这里，我们就得回到全手工时代了吗？这人啊，由俭入奢易，由奢入俭难。我这已经习惯了工业化的生产效率，那是真的不想回去搞纯手工。

那怎么办呢？为什么我们为什么不自己做一个呢？很多人觉得这个是不是太难了？其实在各种框架充斥这个世界之前，我们做开发都是先自己搞一个框架。我们需要思考一下框架本身到底有什么价值，作为开发必须使用的框架，其实承载了两个职责：

- 一个是上下文的切分，比如spring mvc里划分了Model、View和Controller
- 二是提供了开发者实现起来或困难或麻烦的功能。

而因为有第二个职责，所以框架在一直更新，提供所谓更好地接口来方便开发者开发。在以前，更新可以带来成本的优势，而在未来，更新无法带来成本的优势。那么我们只需要写一个简单的框架完成第一个职责就可以了，那这个框架必然很简单。而所谓开发者实现起来困难或麻烦的功能，其实对大模型来说大都不是什么大事。所以自己做的框架更容易甩脱已有框架的历史包袱，对大模型实现更为友好。（说句题外话，其实对于颠覆性创新者来说，这是难得的历史机遇期，因为这种情况下最容易出现所谓“创新者的窘境”。而这次的窘境是贯穿所有领域的，给了颠覆性创新者数不清的机会。）

## 实现

想清楚了我们不需要LangChain而是要自己做一个框架，那么我们就可以自己来写一个。而这个实现是令人法治的简单，因为本质上，我们上一篇讲的流程就是最符合pipeline架构的一种场景，那么我们只需要写一个方便搭建pipeline架构的框架实现就可以了，但是这个要让LLM自己生成。为什么呢？那需要讲一个LLM使用技巧的知识点。

围绕LLM，我们可以把相关知识分成三种：

- 已经被训练进LLM的知识
- 放到LLM提示词里的知识
- LLM生成的知识

前两者应该是比较好了解的，为什么要说第三种知识呢？因为在我们使用的过程中发现，用LLM生成的知识，可以更有效的命中已经被训练进LLM的知识从而提升LLM的表现，也就是减少幻觉。

所以如果我们希望大模型可以用好一个框架，最好是给大模型他自己写的框架。所以为了完成这个管道架构，我让大模型给我写了一个管道架构的实现：

```js
class PipelineComponent {
    async run(input) {
        throw new Error('Run method must be implemented');
    }
}

class Pipe extends PipelineComponent {
    constructor(func) {
        super();
        this.func = func;
    }

    async run(input) {
        try {
            return await this.func(input);
        } catch (error) {
            throw error;
        }
    }
}


class Pipeline extends PipelineComponent {
    constructor() {
        super();
        this.components = [];
    }

    add(component) {
        if (component instanceof PipelineComponent) {
            this.components.push(component);
        } else {
            throw new Error('Component must be a Pipe or Pipeline');
        }
        return this;
    }

    async run(input) {
        let result = input;
        for (let component of this.components) {
            result = await component.run(result);
        }
        return result;
    }
}

const empty_pipe = new Pipe(async(input) => input )

class ConditionalPipe extends PipelineComponent {
    constructor(conditionFunc) {
        super();
        this.conditionFunc = conditionFunc; // 为什么要用function，而不用pipe，因为pipe有个职责是要维护input这个context持续向后传，而我们希望这个function不要有赋值动作，不要有这个义务，返回true/false
        this.trueComponent = empty_pipe;
        this.falseComponent = empty_pipe;
    }

    setTrueComponent(component) {
        if (component instanceof PipelineComponent) {
            this.trueComponent = component;
        } else {
            throw new Error('True component must be a Pipe or Pipeline');
        }
        return this;
    }

    setFalseComponent(component) {
        if (component instanceof PipelineComponent) {
            this.falseComponent = component;
        } else {
            throw new Error('False component must be a Pipe or Pipeline');
        }
        return this;
    }

    async run(input) {
        if (!this.trueComponent || !this.falseComponent) {
            throw new Error('Conditional pipe requires both true and false components');
        }

        try {
            const conditionResult = await this.conditionFunc(input);
            if (conditionResult) {
                return await this.trueComponent.run(input);
            } else {
                return await this.falseComponent.run(input);
            }
        } catch (error) {
            throw error;
        }
    }
}

class LoopPipe extends PipelineComponent {
    constructor(conditionFunc) {
        super();
        this.conditionFunc = conditionFunc; // 设置循环条件
        this.loopComponent = null;
    }

    setLoopComponent(component) {
        if (!(component instanceof PipelineComponent)) {
            throw new Error('Loop component must be a Pipe or Pipeline');
        }
        this.loopComponent = component;
        return this;
    }

    async run(input) {
        if (!this.loopComponent) {
            throw new Error('Loop component is not set');
        }

        let result = input;
        while (await this.conditionFunc(result)) {
            result = await this.loopComponent.run(result);
        }
        return result;
    }
}

module.exports = {
    Pipeline,
    Pipe,
    ConditionalPipe,
    LoopPipe
};
  
```

代码很短，才100来行，基本上就能实现简单的顺序、选择、循环了。自己做一个流程很简单了。具体用起来大概这样，可以看出来，能够很简单的在更高的抽象层次上进行面向过程编程：

```javascript
const pipeline = new Pipeline();
pipeline.add(check_token_limit_pipe)
    .add(new ConditionalPipe(over_token_limited)
            .setTrueComponent(new Pipeline().add(assemble_chat_history_pipe)
                                .add(generate_core_memory_pipe)
                                .add(simple_string_merge_core_memory_pipe)
                                .add(load_all_core_memory_pipe)
                                .add(based_on_core_memory_prepare_system_prompt_pipe))
    .add(transform_to_openai_messages_pipe);
    .add(openai_send_messages_pipe);
```

而整个pipeline框架的实现方式因为不涉及其他上下文，整个过程就是最原始的许愿驱动开发，下面是我的全部提示词，一共九次对话就完成了：

```
我想设计一个pipeline架构的nodejs框架，支持可以把多个pipe串在一起。

我希望每一个pipe处理完之后，可以有不同的状态，根据不同的状态选择不同的后续pipeline

不对啊，pipeline里应该是可以放入 pipe 和 pipeline，然后把pipe和后续pipeline可以串起来

如果我希望存在某种if pipe，可以设置后续执行哪个pipe或pipeline

我希望coditionpipe的初始化和设置true/false走哪个pipe或者pipeline分离，

如果我想加入一个循环pipe怎么做，可以根据结果判断要不要循环？

不是，我希望的loopPipe里面执行的也是一个pipe或pipeline

我希望循环的判断函数初始化和循环体分离

我觉得应该先设置condition，然后加载循环体
```

这个跟前面讲的许愿驱动开发是一样的，只是它的上下文在前面的对话里，接下来我们只需要给出那个框架代码，配上项目里的代码，接着许愿让他写pipe就可以了。而因为我们的pipeline框架非常的简单，所以放到提示词里LLM理解起来也很容易。尽管我们的代码极其简单，几乎没有什么约束，但是LLM一般很听话，你要求他做的行为他一定会做，比如保持传入参数作为传出数据，所以很多约束根本没有必要通过框架本身限制。而所谓框架提供的生态，其实相当多的功能让LLM给你重写一遍，也不花多少时间，而且由于只需要关注你的特殊需求而不用考虑通用需求，它的代码可能还更简短。

当你真的这么做的时候，你会发现，他可以跟我们各种实践很好的组合。任何一个外部库你可以封装一个pipe，只要你写个大概的jsdoc，让LLM知道输入输出，它其实可以很好的使用已经写好的pipe。另外通过我们前面pipeline的组合代码，我们也可以通过静态分析，更精密的加载所需的pipe代码而不是所有的代码。

所以在今天这个时代，如果我们要做一些Agent的Flow，完全没有必要使用一些重型的框架引进来，从这个角度讲，可能一些轻型框架更容易崛起。

题外话：公众号没法对话，各个平台我发的文章我也没有时间一一关注后续，所以我搞了一个知识星球，想要问我问题的人可以加入我的知识星球。另外，有一些实操性质的内容，比如具体的怎么用提示词写代码的题目和参考答案、我怎么用ChatGPT写的这些库的过程、以及我这些年搞过的培训的练习题和参考答案以及一些其他的内容吧，放到公开渠道上也没啥人看，准备统一放到我的知识星球里了，感兴趣的可以加入进行实操层面的学习。（假如“铜剑技校”是个门派，那发在公开渠道的就算外门，知识星球就算内门:-P）
