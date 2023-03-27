---
title: ChatGPT编程秀-5： 测试驱动ChatGPT编程
date: 2023-3-17 18:12
---

## 有输入就要有输出

上一篇文章中，我故意漏掉了一个手法没有讲。具体是什么样的手法呢？其实在实施的过程中，我发现把主干流程的逻辑讲的再清楚，他生成的时候还是会有很多错误，改进自己的描述已经让我觉得有些烦躁了。我不由得想起了2023年1月，ECM发了一篇文章：《The End of Programming》以呼应ChatGPT的诞生，在文章的最后写道:

> 我们正在迅速走向这样一个世界：计算的基本构件是有脾气的、神秘的、自适应的代理。  

好家伙，克苏鲁神话的味都出来了，世界的底层是混乱与疯狂是吗？所以ChatGPT就是是活化的隐匿贤者？^_^

玩完梗我们回来看这个事情啊，突然我意识到，是不是我之前的prompt还缺了一些东西？我只给了输入和主干逻辑，我没有给他输出啊。在我的视角里，可能这个输入通过这个主干逻辑只能有一种结果，但是对于AI来说，也未必啊（别说AI了，我跟另一个初级开发这么讲，他都未必能写出一种结果来，只能说这个行为表现太人类了）。如果我把输出也给他是不是可以让他写的更好一点，于是我把我的prompt改成了下面的描述：

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
      - chara: #  这里改成了数组
        - Abyssinian,
        - cat_in_boots,
      facial_expressions:
        - (smile:1.5),  
        - (smile:1.2),  
        - smile, 
    steps: 20

```
>可能的输出：  
``` js
[
{
    steps: 20,
    prompt: 'a cat,\nAbyssinian,\n(smile:1.5),\n',
    batch_size: 1
},
{
    steps: 20,
    prompt: 'a cat,\nAbyssinian,\n(smile:1.2),\n',
    batch_size: 1
},
{
    steps: 20,
    prompt: 'a cat,\nAbyssinian,\nsmile,\n',
    batch_size: 1
},
{
    steps: 20,
    prompt: 'a cat,\ncat_in_boots,\n(smile:1.5),\n',
    batch_size: 1
},
{
    steps: 20,
    prompt: 'a cat,\ncat_in_boots,\n(smile:1.2),\n',
    batch_size: 1
},
{
    steps: 20,
    prompt: 'a cat,\ncat_in_boots,\nsmile,\n',
    batch_size: 1
},  
]
```
		    
>要求：  
>1. 假设上面的yaml转成json的转换代码我已经写完了
>2. 我需要遍历poly下的所有的顶层元素
>3. 遍历过程中，要处理template_prompt元素的子元素：
>    1. 从template中读取作为模版。
>    2. 读取meta中的属性，因为属性可能每次都不一样，是不确定的，所以不能硬编码。
>    3. 然后基于meta中的属性，把template作为 string literal 解析，这个解析代码我已经有了，假设名为render_string_template，可以不实现，留一个函数接口即可。
>    4. 要遍历组合meta中的每一个属性组形成一个数组，
>        1. 每一个属性组可能只需要看做一个对象，当且仅当每一个属性值都为单值
>        2. 每一个属性组可能也需要展开，当且仅当任何一个属性值有多值，比如 facial_expressions 有一个值，chara有两个值，那么应该生成1*2也就是两组属性放入这个数组中，这个数组和template会被传入render_string_template函数，最后会获得两个prompt字符串
>4. 将生成的个prompt字符串数组和template_prompt元素之外的其他元素合并成一个对象，要求在同一级别。prompt字符串数组有几个元素，就会合并成几个对象，并放入一个新数组中，我们称之为ploys。
>5. 继续遍历，直到遍历完所有顶层元素，所有元素都放入了polys中。polys是一个一维数组。
>6. 将ploys中的每一个元素与base中的属性合成一个新的对象，base的属性展开与prompt属性同级，当ploys中的每一个元素的属性名与base中的属性名相同时，覆盖base中的属性。这些新对象组合出的数组就是我要的数组

果然就得到了预期的结果。

这一个动作，让我打开了思路，用输入+输出框住它生成的边界还是挺好用的。输入+输出框住边界？这不就是测试吗？

## 停下来想一想

从我们的体验来看，确实啊，ChatGPT生成的是有点不稳定。《The End of Programming》说的没错，底层确实有点混乱与疯狂的味道，起码不太稳定。但这事也就听起来很吓人，说实在的，人就比ChatGPT稳定多少呢？我这个人比较粗心大意，我写代码的时候也经常脑子一抽，写出一些事后看自己都想抽自己的脑残错误，所以我自打听说了TDD，很快就变成了坚定地TDD原教旨主义者，没有TDD的世界对我们这种人来说本来就是混乱与疯狂的，要说驾驭软件开发过程中的混乱与疯狂，那你是问对人了。

那么回顾一下TDD是什么？下面是一个复杂版
![tdd-process-complex-version](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/chatgpt-coding-show/05-test-driven-chatgpt-development/05-01-tdd-process-complex-version.png)

基本上就是，先写一个测试用例，然后执行，得到期望的失败，什么是期望的失败呢，比如说，你写了一个add函数，接受两个参数，然后你写了一个add(1,1)，你期望的失败可能是返回某个值，他不等于2，实际你执行的时候呢，报错，说add函数不存在，这就不是你期望的失败。你要调整环境到你期望的失败，然后开始写实现，写完实现再执行，如果测试不通过了，就接着改实现，直到通过。如果通过了，就问自己要不要重构，如果要重构，就改实现，改完的还要通过测试，如果不想重构了，那就写下一个测试用例。总之就是这么个往复循环的流程。

如果上面那个复杂版让你觉得很晕呢，可以看这个简单版：
![tdd-process-simple-version](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/chatgpt-coding-show/05-test-driven-chatgpt-development/05-02-tdd-process-simple-version.png)

红表示测试执行失败，绿表示测试执行通过，重构就不用解释了。所以永远是绿了重构，重构完可能红可能绿，红了就让他变绿。只有变绿后，可以开始下一次循环。

上面这些图讲的是TDD的过程，我个人觉得更核心的是，在实现代码的这个过程中，你同时生产了一套自动化测试，他可以在你将来不管是加新功能还是修Bug还是重构，都不会陷入按下葫芦起来瓢的窘境。那么在ChatGPT的时代，这个方法还能用吗？总感觉哪里需要修改一下，不过没关系，我们先走起来，边走边想。

## 想好了，继续出发

那么既然是TDD，应该先写测试，但是我实现都写完了啊。算了，这也是大多数项目的现状，那就先补个测试吧。但是补测试是个很烦的事情啊，我灵机一动，要不……让ChatGPT给我搞？它能行吗？谁知道呢，试试又不花钱。
于是我就说：

> 给下面代码设计一些测试用例：
``` js
module.exports = function render_my_yaml_dsl(data) {
    const vm = require('vm');
    // 定义一个函数用于渲染字符串模板
    function render_string_template(originalScriptTemplate, intention) {
        // console.log(intention);
        // 省略渲染代码
        const execScriptTemplate = "`" + originalScriptTemplate + "`";
        const script = new vm.Script(execScriptTemplate);
        const execScript = script.runInNewContext(intention);
        // console.debug(execScript);
        return execScript;
    }
    // 定义一个函数用于展开属性组合
    function expand_attributes(attributes) {
        const result = [];
        attributes.forEach(obj => {
            Object.values(obj).forEach(val => {
                if (Array.isArray(val)) {
                    val.forEach(v => {
                        result.push({ ...obj, [Object.keys(obj).find(key => obj[key] === val)]: v });
                    });
                } else {
                    result.push(obj);
                }
            });
        });
        return result;
    }
    const polys = [];
    for (const poly of data.poly) {
        const { template_prompt, ...other } = poly;
        const { template, meta } = template_prompt;
        const variableGroups = expand_attributes(meta);
        const prompts = variableGroups.map(variables => render_string_template(template, variables));
        for (const prompt of prompts) {
            polys.push({ prompt, ...other });
        }
    }
    const result = polys.map(poly => ({ ...data.base, ...poly }));
    return result;
}
```

是的就是这么任性，代码直接扔给他。然后我得到了下面的回复:

> 以下是一些可能的测试用例：
> - 当输入参数 data 为空对象时，函数是否能正常运行并返回一个空数组？
> - 当输入参数 data.poly 为空数组时，函数是否能正常运行并返回一个空数组？
> - 当输入参数 data.poly 不包含任何属性组合时，函数是否能正常运行并返回一个包含原始数据的数组？
> - 当输入参数 data.poly 包含一个属性组合时，函数是否能正常运行并返回一个包含根据属性组合渲染的数据的数组？
> - 当输入参数 data.poly 包含多个属性组合时，函数是否能正常运行并返回一个包含根据所有属性组合渲染的数据的数组？
> - 当模板中的字符串模板包含模板变量时，函数是否能正常运行并渲染模板变量？
> - 当模板中的字符串模板包含 JavaScript 代码时，函数是否能正常运行并执行 JavaScript 代码？
> - 当属性组合中包含数组时，函数是否能正常运行并正确展开属性组合？
> - 当属性组合中包含对象时，函数是否能正常运行并正确展开属性组合？
> - 当输入参数 data.base 中包含对象时，函数是否能正常运行并将 data.base 和渲染的属性组合合并为一个对象？

我叹了口气，我在期望什么，果然是这么泛泛的片汤话吗？这玩意还用你告诉我？而且，你这里面还有些不是我想要的啊。唉……心中抱怨了一阵后，我还是不死心，我继续说：

> 分别准备一些数据

这个要求仿佛打开了潘多拉的盒子，我得到了下面的输出：

> 以下是可能的数据准备：
> - 1. 空对象：
```js
const data = {};
```
> - 2. 空的 poly 数组：
```js
const data = {
poly: []
};
```
> - 3. 包含一个属性组合的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name}: ${price}',
     meta: [
       { name: 'color', value: ['red', 'blue'] },
       { name: 'price', value: [10, 20] }
     ]
   },
   id: 1
 }
]
};
```
> - 4. 包含多个属性组合的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name}: ${price}',
     meta: [
       { name: 'color', value: ['red', 'blue'] },
       { name: 'price', value: [10, 20] }
     ]
   },
   id: 1
 },
 {
   template_prompt: {
     template: '${name} (${size}): ${price}',
     meta: [
       { name: 'size', value: ['S', 'M', 'L'] },
       { name: 'price', value: [30, 40, 50] }
     ]
   },
   id: 2
 }
]
};
```
> - 5. 包含模板变量和 JavaScript 代码的字符串模板的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name}: ${price + tax}',
     meta: [
       { name: 'color', value: ['red'] },
       { name: 'price', value: [10] },
       { name: 'tax', value: [2] }
     ]
   },
   id: 1
 }
]
};
```
> - 6. 包含数组和对象的属性组合的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name} (${color.name}): ${price}',
     meta: [
       { color: { name: 'red', code: '#ff0000' }, price: [10] },
       { color: { name: 'blue', code: '#0000ff' }, price: [20] }
     ]
   },
   id: 1
 }
]
};
```
> - 7. 包含属性值为数组，但未包含数组元素的属性组合的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name} (${color}): ${price}',
     meta: [
       { color: ['red'], price: [10] },
       { color: ['blue'], price: [20] }
     ]
   },
   id: 1
 }
]
};
```
> - 8. 包含模板变量为属性值的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${props.name}: ${props.price}',
     meta: [
       { props: { name: 'Product B', price: 30 } }
     ]
   },
   id: 1
 }
]
};
```
> - 9. 包含 render_string_template 函数中 console.log 调用的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name}: ${price}',
     meta: [
       { name: 'color', value: ['red', 'blue'] },
       { name: 'price', value: [10, 20] }
     ]
   },
   id: 1
 }
]
};
console.log(render_my_yaml_dsl(data));
```
> - 10. 包含 expand_attributes 函数中未包含数组的属性组合的数据：
```js
const data = {
base: {
 id: 1,
 name: 'Product A'
},
poly: [
 {
   template_prompt: {
     template: '${name} (${color}): ${price}',
     meta: [
       { color: 'red', price: [10] },
       { color: 'blue', price: [20] }
     ]
   },
   id: 1
 }
]
};
```

这一刻，我仿佛听到了什么出现裂纹的声音，是我的饭碗吗^_^。

我曾经愚蠢的以为，当我们有了ChatGPT这么强大的工具后，我们只需要写测试就可以了，让他去写实现，而我没有发现这个逻辑里面一个愚蠢的漏洞，一个AI都能写实现了，为什么他写不了测试呢？反过来讲，有很多客户问我有没有自动生成自动化测试的工具，因为他们不想写测试，现在也一样尴尬了。好消息是，有，坏消息是，实现他也能写。

激动了一会，我渐渐冷静下来了，没事没事，这测试没法直接用啊，对吧，你看这测试难道还能扔给一个框架让他自动运行吗，那不是还要写一个专用框架………………想到这，我意识到……好像……我在第一篇的时候干过这事……然后我怀着忐忑的心，写下了下面的prompt：

> 我们的文件夹结构如下所示
> ```
> .
> ├── index.js
> └── test
> ├── cases
> │   ├── 1.input.json
> │   ├── 1.output.json
> │   ├── 2.input.json
> │   ├── 2.output.json
> │   ├── 3.input.json
> │   ├── 3.output.json
> │   ├── 4.input.json
> │   └── 4.output.json
> └── index.js
> ```
> index.js 是一个函数，大概如下:
``` js
module.exports = function render_my_yaml_dsl(data){
//....
}
```
> test/cases下是所有的测试数据，input和output为一组，比如1.input.json存储了一个测试的描述和given数据，1.output.json存储了一个测试的expect数据。两者在.input或.output之前的字符串完全相等为一组。
> input.json数据结构为：
```json
    {
    "desc": "测试用例描述",
    "data": { //具体的测试数据
            }
    }
```
> output.json则全部为具体的测试数据
> 基于这个文件夹结构写一个测试框架，放在test/index.js里，要求：
>  1. 可以遍历test/cases下的文件，分组执行且互相不干扰，也就是其中一组失败了也会继续运行下一组，只是会打印出错误信息。
>  2. 可以在上层文件夹执行```node test```来执行
>  3. 读取文件的fs，要使用const fs = require('fs/promise')引入。
>  4. 用assert库进行断言
>  5. 每执行一个测试，要打印一个提示，提示的部分内容要来自input.json 中的desc
> 给出这个代码

然后我得到了下面的代码：


> 可以使用以下代码实现测试框架：
```js
const assert = require('assert');
const fs = require('fs/promises');
const path = require('path');
const render_my_yaml_dsl = require('../index');
const testDir = path.join(__dirname, 'cases');

async function runTests() {
    const files = await fs.readdir(testDir);
    for (const file of files) {
        if (!file.endsWith('.input.json')) {
            continue;
        }
        const caseName = path.basename(file, '.input.json');
        const input = await fs.readFile(path.join(testDir, `${caseName}.input.json`), 'utf8');
        const output = await fs.readFile(path.join(testDir, `${caseName}.output.json`), 'utf8');
        const { desc, data } = JSON.parse(input);
        console.log(`Running test "${desc}"`);
        try {
            const result = render_my_yaml_dsl(data);
            assert.deepStrictEqual(result, JSON.parse(output));
            console.log(`Test "${desc}" passed`);
        } catch (error) {
            console.error(`Test "${desc}" failed`);
            console.error(error);
        }
    }
}
runTests();
```
> 解释：  
> - 首先引入 assert 和 fs 模块，并且导入需要测试的函数 render_my_yaml_dsl。
> - 定义测试文件夹路径 testDir。
> - 创建一个异步函数 runTests 来遍历测试文件夹下的所有测试用例。
> - 使用 fs.readdir 异步读取测试文件夹下的所有文件，然后遍历文件数组，只选择以 .input.json 结尾的文件，跳过其他文件。
> - 通过 path.basename 函数获取测试用例的名称，然后使用 fs.readFile 异步读取该测试用例的输入和输出文件，并使用 JSON.parse 解析 JSON 格式的数据。
> - 在控制台中打印正在运行的测试用例的描述 desc。
> - 使用 try...catch 语句运行测试用例，并使用 assert.deepStrictEqual 函数断言测试结果是否与期望值相等。如果测试通过，打印成功信息，否则打印错误信息。
> - 最后调用 runTests 函数运行测试。
  
这一刻我终于确信，那确实是饭碗出现裂纹的声音。(虽然上面的只有输入，但是后来我让他给出输出，他也给出了，反正测试框架都有了，大不了跑一遍就有了所有的输出了嘛，所以这不是啥大问题。)

有了这个框架之后，我工作流程大概变成了这么个节奏：
1. 告诉他，我要扩展新功能，然后扔给他旧代码，接着告诉他这里是新新功能需要的输入，我期望的输出是什么。边界是什么，现在给我代码。
2. 然后执行新加的测试，
   1. 如果新测试不通过，就让他重新生成.
   2. 如果新加测试通过了，但是旧的测试废了，就把就废了测试配上代码给他，告诉他代码有Bug，这是以前的输入，期望的输出是什么，你现在的输出是什么，让他改代码。
整个过程就很像TDD的红-绿循环，虽然重构没有了，但是红绿循环还是有的。
而更过分的是，一开始新功能需要的测试用例我都懒得自己写，我就大概告诉他要搞个什么样的扩展，给他代码和旧得测试用例结构，让他给我写个新的测试用例。然后就给我写出来了。（也不总能很完美，但是就是需要改也比以前快了不知道多少，关键不用去想那些繁琐的细节也是提供了一定程度的情绪价值。）

按照我的工作流程画个人在回路是这样的：

![chatgpt-tdd-human-in-the-loop](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/chatgpt-coding-show/05-test-driven-chatgpt-development/05-03-chatgpt-tdd-human-in-the-loop.png)

## 总结一下

开篇我们从一个上一篇漏掉的关键手法开始，了解到输入和输出配合可以让ChatGPT写出的代码更靠谱，而且对于主干流程的描述可以不用那么复杂。

接着我们发现，这确定了输入输出就很像测试，那么我们是不是可以用测试驱动的方式驱动ChatGPT开发呢？经过一番尝试我们得到了一个可以用于ChatGPT的类TDD工作方式。并画出了整个人在回路。

这个回路很像TDD，但在这个回路里，我们既不需要写测试，也不需要写实现，我们主要的工作是保证ChatGPT在按照整个TDD的流程在写代码。因为TDD属于XP（极限编程）的核心实践，所以我们开玩笑说，参照Scrum Master，我们以后可以叫自己XP Master。被人提醒Master会被冲，那我们就叫自己 XP Shifu 吧。（典出功夫熊猫^_^）

目前受制于GPT3.5的3000多字的限制，只能一个个用例让他改，等GPT4的3万多字成为常态后，这个工作方法只会更强大，甚至可以考虑某种程度的自动化。因为我们可以看到，人在回路上只有一个环节需要人参与，其他的都可以不需要。这就是我们上篇文章中提到的，可以自动化的一种思路，有想做工具的可以考虑一下，我还挺需要这么个工具的。

整个用ChatGPT编程的思路到这里主干就讲的差不多了，接下来我们会讲一些细分场景的套路。然后如果有时间的话，就把派发引擎和自动化工具也试着做一做，把过程文字直播在这里，感兴趣的可以关注一波：）
