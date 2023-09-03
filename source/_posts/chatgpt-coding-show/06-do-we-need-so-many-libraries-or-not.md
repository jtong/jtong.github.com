---
title: ChatGPT编程秀-6： 我们还需要那么多库吗
date: 2023-3-17 18:12
---

## 瀑布流展示图片的新需求

我已经可以生成一些图片了，但是光看图片我能看出什么来？我需要展示出图片和图片的信息。这可能就要专门开发一下了。

我当时想，这个可能有点难，他是个前后端都有的程序。所以我就本能的想到，要不要找找有个库没有，先找个瀑布流展示图片的现成的库，在上面稍微改改可能是比较简单的做法。害怕ChatGPT不行，我还去问了new bing，我想着，newbing的更新一点，于是我问：

> 指定一个path直接启动一个nodejs server，通过localhost访问，以瀑布流形式展示path下所有图片的npm工具  

结果他建议我使用node-static的库，于是我就追问了一些怎么用node-static库的方式去写代码，得到的代码可能由于知识过时，反正是不能用的，我就不贴在这里了。

有点沮丧，这稍微用点库你就给我撂挑子，你这果然也不行啊，一用库就出错，你这果然胜任不了复杂开发。new bing不靠谱，我就又去问ChatGPT，一开始我问得有问题，我直接问：
> 用node-static库怎么实现：指定一个path直接启动一个nodejs server，通过localhost访问，以瀑布流形式展示path下所有图片  

结果显示的代码果然是不能用的，我心想对啊，new bing都没有新数据，你这个数据岂不是更糟糕？但是灵机一动之下，我也没退出，我就问
> 上面的代码不能运行，有没有别的办法?  

然后我得到了下面的回答：

>可以使用Node.js的http和fs模块来启动一个HTTP服务器，并且使用path模块来获取文件路径，以及使用mime模块来获取文件的MIME类型，然后通过HTML和CSS来创建一个简单的瀑布流布局。	  
>
> 以下是一个示例代码：
	  
```js
const http = require('http');
const fs = require('fs');
const path = require('path');
const mime = require('mime');

const PORT = 3000;
const PATH = './path'; // 设置图片目录的路径

http.createServer((req, res) => {
console.log(`Request for ${req.url}`);

// 如果请求的是根路径，返回图片列表页面
if (req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html' });

    // 读取图片目录下的所有文件
    fs.readdir(PATH, (err, files) => {
    if (err) {
        console.error(err);
        res.end('Error reading directory');
        return;
    }

    // 使用HTML和CSS创建瀑布流布局
    res.write(`
        <html>
        <head>
            <title>Image Gallery</title>
            <style>
            .container {
                display: grid;
                grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
                grid-gap: 10px;
            }

            .item {
                position: relative;
            }

            .item img {
                width: 100%;
                height: auto;
            }

            .item .caption {
                position: absolute;
                bottom: 0;
                left: 0;
                right: 0;
                background-color: rgba(0, 0, 0, 0.5);
                color: #fff;
                padding: 5px;
            }
            </style>
        </head>
        <body>
            <div class="container">
    `);

    // 遍历所有文件，并在瀑布流布局中展示图片和文件名
    files.forEach((file) => {
        const filePath = path.join(PATH, file);
        const mimeType = mime.getType(filePath);

        if (mimeType && mimeType.startsWith('image/')) {
        res.write(`
            <div class="item">
            <img src="${filePath}" alt="${file}">
            <div class="caption">${file}</div>
            </div>
        `);
        }
    });

    res.write(`
            </div>
        </body>
        </html>
    `);

    res.end();
    });
} else {
    // 否则，返回请求的文件
    const filePath = path.join(PATH, req.url);
    const mimeType = mime.getType(filePath);

    if (mimeType && mimeType.startsWith('image/')) {
    // 如果请求的是图片文件，则返回图片数据
    fs.readFile(filePath, (err, data) => {
        if (err) {
        console.error(err);
        res.writeHead(404);
        res.end('Not found');
        return;
        }

        res.writeHead(200, { 'Content-Type': mimeType });
        res.end(data);
    });
    } else {
    // 否则，返回404错误
    res.writeHead(404);
    res.end('Not found');
    }
}
}).listen(PORT);

console.log(`Server running at http://localhost:${PORT}`);
```
> 将上述代码保存为app.js文件，然后在终端中进入该文件所在的目录，并运行以下命令：	  
> node app.js
> 运行该命令后，服务器将会在本地3000端口上启动。您可以在浏览器中输入http://localhost:3000/来查看图片列表，其中/表示根路径，这将会显示图片列表页面，瀑布流形式展示path下所有图片。如果您想直接访问某张图片，可以通过输入其文件名来访问，例如http://localhost:3000/example.jpg。请注意，在上述示例中，我们假设所有图片文件的MIME类型都以image/开头，这通常是正确的，但如果您的图片文件类型不同，则可能需要修改相关代码。
	  
咦？可以用原生写法啊，我立刻修改了自己的提问：
> 我需要用nodejs写一个后端服务，可以指定一个文件夹，按瀑布流显示下面所有的png图片。
> 要求：
> 1. 可以通过传参指定任意基于index.js的相对文件夹及子文件夹
> 2. 要并排显示png图片和png图片文件中的文本信息
> 3. 页面要用BootStrap布局
> 4. 我有一个模块：'./util/png_info/index'。 该模块导出的对象上有一个异步函数 readPng 可以读取 PNG 图像文件中的文本信息。不用实现，其签名为：
``` js
async function readPng(filePath) {
//....
}
```
> 5. 指定index.js所在文件夹为静态文件夹


我得到了可以执行的代码：

``` js
const express = require('express');
const path = require('path');
const fs = require('fs');
const { readPng } = require('./util/png_info');

const app = express();

// 设置静态文件夹
app.use(express.static(path.join(__dirname)));

app.get('/images', async (req, res) => {
try {
    const folderPath = req.query.folder;
    const images = await getImages(folderPath);
    const imagesHTML = await Promise.all(
    images.map(async (image) => {
        const info = await readPng(image.path);
        return `
        <div class="col-md-3">
            <img src="${image.url}" class="img-thumbnail" alt="${image.name}">
            <div class="text-muted">${info}</div>
        </div>
        `;
    })
    );
    const html = `
    <html>
        <head>
        <title>Image Gallery</title>
        <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/5.3.2/css/bootstrap.min.css" rel="stylesheet">
        </head>
        <body>
        <div class="container">
            <div class="row">
            ${imagesHTML.join('')}
            </div>
        </div>
        </body>
    </html>
    `;
    res.send(html);
} catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
}
});

async function getImages(folderPath) {
const images = [];
const files = await fs.promises.readdir(folderPath, { withFileTypes: true });
for (const file of files) {
    if (file.isDirectory()) {
    const subfolderPath = path.join(folderPath, file.name);
    const subfolderImages = await getImages(subfolderPath);
    images.push(...subfolderImages);
    } else if (file.name.endsWith('.png')) {
    const url = path.join('/', path.relative(__dirname, folderPath), file.name).replace(/\\/g, '/');
    images.push({ name: file.name, path: path.join(folderPath, file.name), url });
    }
}
return images;
}

const port = process.env.PORT || 3000;
app.listen(port, () => {
console.log(`Server listening on port ${port}`);
});

```

## 开悟时刻

这一刻，我开悟了，对啊，要什么库啊！我需要重新反思我的工作习惯，以前之所以要搞那么多的库为的是什么？现在这些库还有没有存在的必要要重新思考一下。

比如这里，我觉得写一个瀑布流很麻烦，于是我想到搞个库，这个是一个旧时代的思维定式。这是因为自己开发很花成本，我自己写有可能会相关技术不熟有可能会写错，所以我的生产效率比较低。比如写一个例子里这样的东西，它涉及到非常多的知识点：nodejs的、css的、HTML的；还有很多集成点：前端后端你把它们连起来，可能会连错。太多的点要考虑了。这个每一个点上我可能我的知识储备不够，或者我忘了，我还要重新去学习，重新去查找；又或者我在集成的时候，个人会犯太多的错，完了就要重新调试，都是成本。所以我选择了使用一个库。

但是使用一个库就没有成本吗？查找是个成本，学习是个成本，然后呢你查的资料和现在的库有可能对不上，还会产生浪费，也是成本。但是在以前我们就忍了，因为自己开发的成本远远大于使用库的成本。但是今天，对于ChatGPT来说，前面提到的自己开发的那些问题，它还是问题吗？它不是问题啊。

然后我又想到前面的node-static库为什么不行？而你看在这个里面他也用了库，他用了express，用了fs，用了path。为什么后面这几个库就没有问题，前面那个node-static库就有问题？因为前面个库接口不稳定，ChatGPT学习到了旧版接口的使用方法，而那个库更新了。这么专用的一个库为什么要更新接口？还不是为了让人用的方便。因为可能会有人反馈说你这个接口不好理解，不容易理解。这个你能不能换一个接口？他这一换接口对于用新接口的人可能是舒服的，但是对于像我这样只是想简单用一下的人就很郁闷了，因为资料都变了。很有可能会出现我查了资料，照着写了代码，然后用不起来的情况。在软件开发的世界里面，这种事情不要太多。

对于 ChatGPT 来说，这种变化是好还是坏？人在使用上那一点点微小的\心理上的爽感对于 ChatGPT 来说是没有任何好处的，反而是有害的。那么在今天这个时代，我们为什么还要使用那些繁重的库？我们是不是应该直接回归本源？就用最基础、最简单封装的库。这些都有一个特点。接口稳定。从诞生之初到现在没有太大变化。这个对于ChatGPT才是最友好的。

## 推而广之

让我们畅想一下，比如Java企业开发领域，是不是像是spring data这种东西。它的价值可能就不如SQL了，spring data这个东西一年不看都能有一大堆的变化，而sql这么多年了都没什么变化。这才是最好的。除非我们能找到。SQL不能满足的场景。在那里封装一个简单的框架。比如说就是MyBatis。

所以在基于 ChatGPT 进行编程为前提，我们进行技术栈选型时，依赖库接口的稳定性就变得格外重要了。不客气的说，在今天这个时代，接口的稳定性压倒一切。这就像在蒸汽机时代，铁轨的间距是不能随心随意的。只有稳定的铁轨间距。才会有高效的运输效率。同理，只有稳定的接口才能最大限度的发挥 ChatGPT 的能力，得到极高的开发效率。那些仅仅是因为帮助人少写几行代码而开发的。变化在今天这个时代，能不引入还是不要引入。毕竟你在上面包一层的成本已经开始无限趋近于0了。你为什么还要让下面的接口迁就你呢？下层接口只要稳定的干好自己该干的事情，并且提供稳定的接口就够了。

这个变化对于今天非常多的企业中台都是一个重大利好。我们知道很多企业中台这么多年都疲于奔命。沦为给前台的“大客户”打工。通常对于企业来说，前台分为几种？

一种是类似企业的“大客户”，就是企业里最赚钱的前台部门，那么我们就要配很大的人力去服务好他，“大客户”自己呢，通常就任性一点，所以会把他们的需求直接扔给我。那我们不得不改以适配“大客户”。

还有一些是小客户，但是因为都在企业里，你不能像在市场里一样说，小客户我就不管了，我们是专做高端服务的。小客户你也要管，成年人全都要嘛。这中间的矛盾呢，就使得很多设计上本来已经说的很清楚的原则，根本落了地。

其实想想挺可笑的，过去那么多年，我们都说，下层要稳定下层要稳定，结果下层总也稳定不了。为什么呢？不就是上层的新需求来了，非要让我们下层的搞嘛。该我们搞的，我们搞，你们用着不方便也让我们搞这是不是有点过分了，方便这个东西哪是有止境的呀？最方便的就是你们什么都不做，都让我们做了，才最方便不是吗？这么搞它接口能稳定吗？

但是话又说回来了，以前没有办法呀。中台有很强的开发力量，前台有吗？通常前台是没有的。而前台一句你不好用，中台就政治不正确了，所以不得已我们就要配很多的开发力量来解决这些非技术的所谓技术问题。毕竟大家都要为效率低头嘛，前中后台端到端全局效率最高才是我们追求的方向。你不能说我中台设计的好了，我效率高了，你整体效率上不去。你这个单点效率高又有什么用？

所以这就是个死局。以前大家都装作努力在解，其实本质上都是非常原始的解法，解得所有人都痛苦。但是在今天我们迎来了新的生产工具。让我们看到了这个矛盾可以被消解的可能，因为稳定。不仅仅可以让中台的开发效率提升，也可以让前台开发效率提升。一个变动的接口，对前台也是不友好的，毕竟随着 ChatGPT 这类大语言模型的推广，前台的开发能力被史诗级加强，我们可以看到，微软在Power Platform上已经推出Copilot功能来实践这种模式了。

## 总结一下

开篇我们引入了一个以瀑布流形式展示图片需求，一开始，我们采用习惯的思路，尝试使用一个库来解决这个问题。但马上就遇到了问题，这并不是ChatGPT擅长的事情，然而当我问 ChatGPT 有没有其他方法的时候，我得到了原生写法的解决方案。

这让我们意识到在基于 ChatGPT 编程的背景下，重新审视库。过往的很多库都是会为了人们好用而不停地提供一些新的接口，这些接口让人们用着很舒服，但是其实带来了学习成本，以及学习资料的过时。很多人都不喜欢学习新技术，就是觉得乱，看起来也不是没有道理。

所以那些古老的但是依然在用的库是更能发挥出 ChatGPT 的效力的。依然在使用表示没有过时，依然有价值，古老表示接口稳定，资料最全。所以我们是不是要反思一下，发明那么多库有没有可能一定程度上走错了路。这让我想起三体里面，人类建了那么多的星际舰队，三体就搞了一个水滴，只用撞击这一个动作就把那些花里花哨的攻击方式给比下去了。这层顿悟导致我之后一段时间在技术群里看到有人问什么功能有没有现成的库的时候，不自觉的会在脑子里蹦出一个吐槽：“哼，原始人”。这心态很不好，但是确实很贴切，兴许随着此类大语言模型AI的发展，我们以前的很多花里胡哨的做法真的看起来就很像传统武术里的很多招数，在热武器面前不再具有普遍的价值。

那么推而广之，稳定的接口对于发挥ChatGPT的能力以提高开发效率至关重要。在技术栈选型时，稳定性应该成为优先考虑的因素。甚至于对于中台建设都有很大利好，在企业中台的建设中，前台和中台之间的矛盾导致中台的接口和功能都无法稳定。然而，随着ChatGPT这类大语言模型的普及，前台的开发能力得到了极大的提升，让中台稳定成为可能，微软的Power Platform已经在摸索类似的模式了。

这一篇我们讲了一个思路上转变，使用 ChatGPT 的过程中，经常会遇到这种思路转变的情况，每次转变都会觉得上一刻的自己是多么的愚蠢。所以建议大家尽快用起来，认知的转变可能会带来完全不同的风景。下一篇我们讲讲另一个引起我思路转变的手法，这也是很多人用ChatGPT处理长内容时的一个思维盲区。

