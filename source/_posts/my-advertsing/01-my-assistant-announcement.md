---
title: 这个中秋节，我用许愿驱动开发方法打造了一个本地版POE
date: 2024-09-17 00:49
---

中秋佳节，月圆人团圆。然而今年，我选择了一种特别的方式来庆祝 - 用代码与AI谱写属于自己的"数字团圆"故事。通过**许愿驱动开发**这一创新方法，我打造了一个名为**My Assistant**的本地版POE。

## My Assistant: 打破AI平台束缚的私人助手

**My Assistant**的诞生，源于一个我对AI未来的想法。随着AI技术的普及，我觉得AI服务的成本将大幅降低。在这样的未来，我们每个人都可以拥有自己的AI客户端，只需购买API就能自由地创造各种功能。这种方式不仅经济实惠，更重要的是，它让我们摆脱了被大型AI平台绑架的困境，给我们更多自由。

在这个愿景的驱动下，我创造了My Assistant - 一款为Visual Studio Code量身定制的插件，集成了智能聊天助手和任务管理功能。它不仅是一个工具，更是一种新的AI使用范式。通过My Assistant，用户可以真正掌控自己的AI体验，自由地安装社区开发的各种功能，甚至可以让AI为自己量身定制新功能。

只需一个AI服务的API密钥，你就可以在本地环境中驾驭各种社区开发的Agent，与AI展开深度对话，同时确保所有数据和交互记录都安全存储在你的设备上。这不仅保护了隐私，也为AI应用开辟了无限可能。

### My Assistant的核心优势

1. **真正的自由**: 不再被特定AI平台限制，自由选择和组合各种AI服务。
2. **隐私至上**: 所有数据和对话历史都存储在本地，杜绝信息泄露之忧。
3. **无限可能**: 支持社区贡献的Agent，甚至可以让AI为你定制新功能。
4. **经济实惠**: 随着AI服务价格下降，这种模式将越来越经济实惠。
5. **灵活定制**: 在`ai_helper/agent`目录下，你可以自由加载和定制各种AI Agent，满足个性化需求。
6. **简单上手**: 只需配置AI服务的API密钥，即可开启智能辅助之旅。


## 安装指南

### 前置条件

- **Visual Studio Code**：请确保已安装最新版本的 VSCode。
- **Node.js**：插件依赖于 Node.js，请确保已安装 Node.js 环境。（主要是 agent 需要）

### 安装步骤

1. **搜索插件**

在vscode extensions marketplace搜索 my-assitant ：

![search-plugin](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/my-advertising-01-my-assistant/01-search-plugin.png)

点击 install 即可。

## 配置 API 密钥

插件依赖于不同的 AI 服务来提供智能功能。在使用之前，请确保在 VSCode 的配置中添加相应的 API 密钥。

1. **打开设置**

点击左下角的齿轮图标，选择 `设置`，或者使用快捷键 `Ctrl+,`。

2. **找到插件配置**

在设置搜索栏中输入 `myAssistant.apiKey`。

3. **配置 API 密钥**

根据您使用的 AI 服务，添加相应的 API 密钥：

![setup_apikey](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/my-advertising-01-my-assistant/02-setup_apikey.png)

## 使用指南

### 聊天助手

#### 新建聊天

1. **打开聊天侧边栏**

点击活动栏（通常在 VSCode 左侧）中的聊天图标，打开聊天侧边栏。

2. **创建新聊天**

在聊天列表的标题栏中，点击 `+` 按钮。

1. **选择 Agent**

输入聊天名称，然后从可用 Agent 列表中选择一个 Agent 。Agent 列表来自于项目目录下 `ai_helper/agent/agents.json` 中的配置，您可以根据需要选择适合的 Agent。

2. **开始对话**

双击聊天名称，打开聊天窗口。您现在可以在输入框中输入消息，与 AI 助手进行交互：

![03-chat-view](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/my-advertising-01-my-assistant/03-chat-view.png)

#### 管理聊天

- **重命名聊天**：右键点击聊天名称，选择 `Rename Chat`，然后输入新的名称。
- **删除聊天**：右键点击聊天名称，选择 `Delete Chat`，确认删除该聊天。

### 自定义 Aegnt

`ai_helper/agent` 组件允许您自定义和加载不同的 AI Agent。

#### 体验样例 Agent

要快速体验样例 Agent （仓库地址： [https://github.com/jtong/my_assistant_agent_examples](https://github.com/jtong/my_assistant_agent_examples)），请按以下步骤操作：

1. 在任意目录中打开 Visual Studio Code。

2. 打开终端（Terminal），并执行以下初始化命令：

```bash
git clone https://github.com/jtong/my_assistant_agent_examples.git ai_helper/agent
cd ai_helper/agent
npm install
```

3. 在 Chat List上点击刷新按钮，即可重新加载所有Agent。或者在 VSCode 中重新加载窗口，或者重启插件，使新的 Agent 生效。

#### 添加新的 Agent

1. **编辑 Agent 配置**

Agent 配置文件位于 `ai_helper/agent/agents.json`。打开该文件，按照格式添加新的 Agent 配置。例如：

```json
{
    "name": "YourCustomAgent",
    "path": "./yourCustomAgent.js",
    "metadata": {
    "llm": {
        "apiKey": "yourApiKeyName",
        "model": "your-model-name"
    }
    }
}
```

2. **编写 Agent 代码**

在 `ai_helper/agent` 目录下，创建 `yourCustomAgent.js`，并按照 Agent 模板实现必要的方法。

3. **加载 Agent**

在 Chat List上点击刷新按钮，即可重新加载所有Agent。或者在 VSCode 中重新加载窗口，或者重启插件，使新的 Agent 生效。


### 后续开发

我后续会支持Job类的界面，可以用于非即时性的workflow来执行任务：

![job-view](https://jtong-pic.obs.cn-north-4.myhuaweicloud.com/my-advertising-01-my-assistant/04-job-view.png)

(目前正在开发中……)

## 许愿驱动开发：AI时代的编程魔法

在打造My Assistant的过程中，我全程采用了**许愿驱动开发**方法。这不仅仅是一种开发方式，更是AI时代编程的新范式。通过与AI的对话，我们可以将想法直接转化为代码，就像在代码世界中施展魔法。

如果你对许愿驱动开发感兴趣，想要深入了解这种神奇的开发方式，我强烈建议你去阅读我的《许愿驱动开发》系列文章。在这些文章中，我详细介绍了许愿驱动开发的原理、技巧和实践经验。

对于那些喜欢更直观学习方式的朋友，我正在B站上持续更新一系列演示视频。这些视频不仅展示了许愿驱动开发的实际操作过程，更重要的是，它们旨在指导你如何一步步打造出自己的简化版My Assistant。

我的目标是让每个观看视频的人都能够跟着教程，亲自体验许愿驱动开发的魅力，并最终创建出属于自己的AI助手。这不仅是学习一项新技能，更是参与到AI时代的创新浪潮中。

在后续的内容中，我还会介绍如何使用许愿驱动开发来创建自定义Agent。这意味着你不仅可以使用My Assistant，还可以为它开发独特的功能，真正实现个性化的AI助手体验。

无论你是编程新手还是经验丰富的开发者，许愿驱动开发都可能为你开启一个全新的编程世界，让我们一起探索AI辅助编程的无限可能！

祝大家中秋快乐，也祝愿每个程序员都能在AI的陪伴下，开启一段充满创意和可能性的编程之旅！