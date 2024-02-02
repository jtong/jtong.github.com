---
title: 许愿驱动开发（三）：GUI
date: 2024-01-28 15:20
---

## 背景介绍
上回书说到，我们搞了一个**prompt-context-builder**库，通过``prompt_builder``命令来生成上下文以实现许愿驱动开发。这个玩法呢，虽然可以用了，但是体验不好，所以说我们想搞一个vscode 的plugin，这样我们就有GUI了。

其实吧，在我最早想到许愿驱动开发这个想法去跟朋友聊的时候，朋友就问我为什么不写一个vscode插件或者idea插件呢？我叹了口气，我不上清华北大是我不喜欢吗？那不是不会吗？学也是要花时间的嘛，一想到要学这个东西又要花不少时间我就没有动力开始。然而不得不说，这个时代真的是幸福，用ChatGPT我完全不需要会，所以之前我觉得花时间完全是旧时代的思维惯性造成的习惯性恐惧，后来孩子生病在医院陪床的时候，大段的时间没什么事干，就试着用ChatGPT写了一下，没想到断断续续写着，两三天就做出来了第一版。

这个时代对于想做点东西的人来说真的是幸福啊，一个简单的点的工具纯靠一步步许愿就能完成了。比较起来，怎么把软件设计的更容易被LLM实现反而变得更重要了，这个我们以后找机会慢慢聊。

## 用法介绍

先去vscode上搜索插件：**prompt-context-builder-plugin**，也可以直接访问这个URL ： [https://marketplace.visualstudio.com/items?itemName=jtong.prompt-context-builder-plugin](https://marketplace.visualstudio.com/items?itemName=jtong.prompt-context-builder-plugin)

安装后就可以在左侧看到我们的图标：

![sider-bar-icon](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/03-wdd-gui/01-sider-bar-icon.png)

点击图标后我们就可以看到三个子view：一个Explorer，一个RECENT FILES，一个TEMPLATE FILES，如下所示：

![all-views](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/03-wdd-gui/02-all-views.png)

其中Explorer里面是我们的文件夹结构，TEMPLATE FILES里是我们可用的模板引擎，点击就能打开。接下来我们打开一个模版，然后选择几个文件如下，类似这样：

![select-files](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/03-wdd-gui/03-select-files.png)

然后按下Ctrl+Shift+P（mac下是CMD+Shift+P）弹出窗口：
![04-gen-related-files-command](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/03-wdd-gui/04-gen-related-files-command.png)

选择”Related Files“就会生成相关的文件填在光标处，而选择的文件会出现在RECENT FILES里：

![05-gen-output-files-result](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/03-wdd-gui/05-gen-output-files-result.png)
然后我们再按Ctrl+Shift+P（mac下是CMD+Shift+P）执行命令”Generate Prompt Output“

![06-gen-prompt-command](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/wish-driven-development/03-wdd-gui/06-gen-prompt-command.png)

就输出到了指定的输出目录，然后就可以拷到浏览器里去问openai了。
那么什么是指定的输出目录呢？其实这里面涉及到了一个配置文件问题。

## 配置文件介绍
不同于之前的命令行工具，我们插件要求config.yml必须存在于项目文件夹的根目录下（如果您觉得想改个名或者换个地方，目前还不支持，不过项目反正也开源了，您要不自己许个愿？地址在这：[https://github.com/jtong/prompt_builder_vscode_plugin](https://github.com/jtong/prompt_builder_vscode_plugin)）
然后里面的内容可以参考下面的：
```yaml
project:
  base_path: ./
  ignore:
    path:
      - target
      - .idea
      - .mvn
      - prompt
      - prompt-builder
      - node_modules
      - ai_helper
      - spike
      - doc
      - .git
      - config.yml
      - package-lock.json
    file:
      - .DS_Store
input:
  prompt_template:
    path: ai_helper/prompt_builder/prompt    
  relative_files:
    template: >
      ```yaml

      {{{content}}}

      ```
output:     
  prompt:
    path: ai_helper/prompt_builder/output/
```

可以看出来project还是命令行的那些属性，path是匹配全路径的起始字符串，file则是文件名匹配。而我们前面的Explorer显示文件的时候，会自动过滤掉要求ignore掉的内容。

而input和output顾名思义，用于匹配输入和输出的相关属性，含义如下：

- /input/prompt_template/path ：就是TEMPLATE FILES里显示的模版，读取这里配置的路径下的文件，目前只读一级，不会读取下级文件夹。
- /input/relative_files/template ：是我们执行 Related Files 命令的时候，生成的文本要套什么模版输出，其实这里主要是因为我习惯用markdown文件，如果套个code block会显示高亮，如果你不需要，其实不用加这个code block也能识别。
- /output/prompt/path ： 输出路径地址，也就是上面我们执行 Generate Prompt Output 命令的时候输出文件的目标地址。

上面的这些路径，都是相对于打开的项目文件夹的根路径。

## 可以不可以更给力一点呢？

从上面的介绍可以看出来，我这里做的就是一个方便用户手动选择文件然后生成上下文以进行许愿驱动开发的小工具。我自己用着还挺舒服，因为实际开发过程中，其实一段时间内，上下文并不会频繁变动，在完成一个较大的任务之前，这上下文真的是很固定。

但是肯定会有人说，可不可以更给力一点呢？ 可不可以根据许的愿望，通过文件树和项目里的文件内容，让LLM自己生成愿望相关的上下文呢？这就属于Agent范畴的需求了，我们下一篇来讲讲这种可能性。

