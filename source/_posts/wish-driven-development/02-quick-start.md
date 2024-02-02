---
title: 许愿驱动开发（一）：导入介绍
date: 2024-01-27 20:20
---

## 背景介绍

在上一次的许愿驱动开发的思路被我想出来之后，我先是做了一些试验，正好有个机会要给一个客户的毕业生培训模拟项目准备个周边用户服务代码。于是我就完全尝试了纯手工来进行，果然这个思路是可以写完一个用户用户服务的代码的，只是每次手工拷贝相关代码实在是累。

所以既然手工可行，那么只需要有一个小工具把手工的这部分工作自动化，不就可以很高效的编程了。于是我开始编写一个小工具完成这部分自动化的工作。所以有了这个项目：[prompt-context-builder](https://www.npmjs.com/package/prompt-context-builder)

这个项目名字是不是很简单粗暴？就是构建提示词里的context部分，方便用户只需要许愿就行了，保证许的愿可以在这个上下文下没有歧义。（其实这个项目本身就是许愿驱动代码做出来的，我先手动拷贝粘贴完成了简单的功能。然后用nodejs写了一个简单代码执行代码，就可以得到一个乞丐版的生成提示词的功能了，然后靠这个生成提示词完成了这个项目，完全自举。）

## 用法介绍

那么怎么用呢？首先我们要准备一个提示词模版。这个提示词模版里面我已经支持了几个标签，我一个个给大家讲。

### folder_tree 标签

按照之前说的，我们需要先得到文件树，这个是最大的上下文，我们人也是看到需求后去文件树里面找或者干脆就在脑子里回忆相关文件。所以呢，我提供了一个标签：``{{folder_tree}}``，这是 [handlebars](https://handlebarsjs.com/) 的语法，具体可以参考handlebars的 [文档](https://handlebarsjs.com/guide/) 。

具体怎么用呢，大概是下面这样：
```markdown
## 技术上下文

我们开发的是一个基于Java的微服务，其工程的文件夹树形结构如下：

\```
{{folder_tree}}
\```
```
(上面的\是不必要的，只是因为很多markdown解析器不支持转义字符而显示出来了）

这样当我们编译整个模板的时候就会得到类似下面的输出：
```markdown
## 技术上下文

我们开发的是一个基于Java的微服务，其工程的文件夹树形结构如下：

\```
.
├── pom.xml
├── prompt_builder
│   ├── output
│   ├── prompt
│   │   ├── config.yml
│   │   └── 1.md 
│   └── README.md
└── src
    ├── main
    │   └── java
    └── test
\```
```

可以看到我们把文件树都显示在了文本里面，那么他是怎么知道文件夹的起点在哪的呢？而且我们也不是所有文件夹都关心，有些文件夹比如存放编译出的class文件的文件夹，.git文件夹等等我们其实是不关心的。是否支持过滤呢？秘密就在这个config.yml里。

我们可以设置一个config.yml，这个config里面是这样的数值：
```yaml
project:
base_path: ./
ignore:
    path:
    - target
    - .idea
    - .mvn
    - ai_helper
    - .git
    - config.yml
    file:
    - .DS_Store
```
可以看到这里面base_path就指定了项目的根目录的路径，这里支持相对路径和绝对路径。至于相对路径相对于什么我们后面说。

而ignore里设置的是希望在生成文件树时希望被过滤掉的路径或者文件：

- path指的是相对于base_path的路径是否以这里面的值开头来匹配，如果路径匹配上就不显示。比如target就是相对于base_path下的target文件夹下的所有文件都被过滤掉不显示。而且也支持匹配文件，比如config.yml表示相对于base_path下的config.yml文件会被过滤掉不显示。
- file指的则是不管在哪个文件夹下，只要是这个文件名，就过滤掉，因为像MacOS，你打开一个文件夹他就给你搞个.DS_Store比较烦，所以就加入了这个能力。

### related_files 标签

那么有了文件树还是不够的，当我说“我想要实现给用户assign role的功能”的时候，我是在做一个后端服务还是在做一个前端UI呢，就算是后端，我是要在Spring MVC的Controller里写还是在Jersey的Resource里写呢？是配置Mybatis还是JPA呢？是用annotation还是xml呢？这一切都要由相关的上下文来收敛这个愿望的含义，所以为了达到我们之前说的许的愿没有歧义，我们需要寻找跟我们这个愿望紧密相关的文件。（当然有些东西可能不是来自于文件的，比如数据库表的定义，一开始可能有些migration脚本可以用，但是随着历史演进，可能还不如连上数据库读个schema的定义来的简单直接，不过目前我们就支持了文件，未来再考虑其他资源）

那么，为了达成这个目的，我们提供了 ``{{related_files}}`` 标签，它有两种用法，不过在实践的过程中，我发现下面这种更常用，那我们就讲这一种好了：

```yaml
{{#related_files_from}}
\```yaml
- path: src/main/java/dev/jtong/training/demo/smart/domain/controllers/UsersController.java
reader: controller
methods:
    - changePassword
- path: src/main/java/dev/jtong/training/demo/smart/domain/controllers/representation/UserVO.java
reader: model
\```
{{/related_files_from}}
```
(上面的\是不必要的，只是因为很多markdown解析器不支持转义字符而显示出来了）

这两个标签之间是以yaml形式提供的一组文本，用来指定相关文件。然后我们编译模板的时候就会去加载相关的文件内容并生成一段三级标题的文本，类似下面这样：
```markdown
### src/main/java/dev/jtong/training/demo/smart/domain/controllers/UsersController.java

\```
// 被读取的 UsersController.java 的代码
\```
### src/main/java/dev/jtong/training/demo/smart/domain/controllers/representation/UserVO.java

\```
// 被读取的 UserVO.java 的代码
\```
```

读取方式呢，需要配置一个reader，目前支持的reader有三个：
- controller：用于读取Java代码中的controller，可以指定读取哪些函数（也就是那个methods属性），会保留所有的field和针对的函数（会保留函数的所有注解）。可以用于针对性的要求读取某个函数。
- model：用于读取Java代码中的 model，会保留所有的field，删除掉所有的setter和getter代码。
- all：读取文件所有内容，不做任何静态分析和预处理。

也就是说，只要我们编写了合理的yaml格式的相关文件数据，我们就可以得到一个让程序员的愿望几乎没有歧义的上下文。如果有，那说明相关文件数据还不够合理，那就去改那个数据。（当然这都是理想情况，实际上你许的愿望还是要精确一点，并且使用代码里的词汇，不然结果可能也会偏的。不过比起没给上下文的时候偏的少了，更多时候是得到一个抽象的任务列表，就如上篇所说那样。）

### 执行

那么是怎么执行才能得到上面这些效果呢？我提供了一个命令。首先要安装prompt-context-builder：

```shell
npm install -g prompt-context-builder
```

然后在你的项目上随便建一个目录，我一般是用ai_helper/prompt_builder/prompt。

然后在这里面建立一个模版文件，比如new-feature.md，贴入上面的文本内容。然后在ai_helper/prompt_builder/prompt执行下面的命令：

```shell
prompt_builder -t new-feature.md -c ../config.yml > output/new-feature-1.md
```

所以可以看到我把config算是放到了prompt_builder文件夹下了，那么这个时候base_path的值应该是什么呢？那就是../../，是的，我们的相对路径是相对于你执行`` prompt_builder ``命令的地方。

最终所有的生成的提示词文本都会打印到控制台，我用>输出到了某个文件里。接着我就可以把文件里的内容拷贝到网页版的ChatGPT或者其他任何一个LLM里复用他们的对话流程就可以了。当然如果你愿意，你也可以自己调API，那实现起来也不难，让ChatGPT给你生成调API的代码也就是几分钟的事情，我这里就更专注于提示词的生成了。反正代码也开源了，你们你可以自己拿去完成最后一步：[https://github.com/jtong/prompt-builder](https://github.com/jtong/prompt-builder)。

你就可以看出来，我们这个搞法啊，不是很方便。我也这么觉得，所以我做完这个东西之后，又做了一个vscode插件。这样我们就可以从IDE上直接操作，就会方便很多。所以想到这里，我就做了一个vscode插件，下一节我给大家介绍一下我写的vscode插件，我们用GUI的方式进行许愿驱动开发。

公众号没法对话，各个平台我发的文章我也没有时间一一关注后续，所以我搞了一个知识星球，想要问我问题的人可以加入我的知识星球。另外，有一些实操性质的内容，比如具体的怎么用提示词写代码的题目和参考答案、我怎么用ChatGPT写的这些库的过程、以及我这些年搞过的培训的练习题和参考答案以及一些其他的内容吧，放到公开渠道上也没啥人看，准备统一放到我的知识星球里了，感兴趣的可以加入进行实操层面的学习。（假如“铜剑技校”是个门派，那发在公开渠道的就算外门，知识星球就算内门:-P）
