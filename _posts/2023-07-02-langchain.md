---
title: What's langchain
tags: 
    - AI
    - ongoing
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->

https://python.langchain.com/docs/get_started/introduction 文档内容摘抄节选

<!--more-->

LLM随着chatGPT的面世爆火了一把，其被认为是一项极富改革性的技术，使开发者可以完成之前无法完成的工作。然而，单独使用LLM由于使用的都是预训练的well-known已知知识，无法创造一个真正powerful的app，除非**你可以将它和其他的知识库或者数据处理进行结合**。

而langchain则是专门为了解决上述问题的一个工具库。它能够帮助带有如下应用场景特性的应用实现其功能：
1. 数据依赖：将其他数据源连接到已有的LLM模型
2. agentic：允许一个语言模型同其所在的特定环境进行交互

而langchain作为一个工具库，又还有如下一些开发上的优势：
1. 高度的组件化：对于其中的任意部分功能，都有相应的抽象接口和样例实现，开发者可以根据需要来选择功能
2. 现成的chains实现：提供了大量的结构化组件实现，这些可以组合成强大的chains来实现特定的高级功能

langchain的开发者认为，其可以在如下6个方面为开发者提供帮助：
1. LLMs和相应的prompts
    
    为LLM提供prompts管理、优化，为不同的LLM模型提供统一接口，以及其他一些通用的面向LLM的工具功能

2. chains

    chains则是跳出单一的面向LLM对话，而是自动生成连续一组面向LLM或其他实际功能的调用。langchain为chains生成提供了一组标准接口，以及和其他实际功能的整合，还为一些常见应用场景提供了chains的实现参考

3. data augmented generation

    基于数据增强的内容生成则是涵盖了特定一组chains，其首先和外部数据先进行交互，获取接下来内容生成步骤的所需数据。例如对长段的内容进行总结，以及针对特定数据的知识问答

4. agents

    agents则是引入LLM，让他自己判断接下来该怎么做。agents帮其完成动作（如访问网页），并将反馈返回给LLM

5. memory

    memory则是指在多个chains/agents调用中的一些持久化存储的信息。langchain提供一组memory接口及其实现，以及一些使用memory的chains/agents实现

6. evaluation

    生成式的模型的一个困难点在于如何评估其性能，一种思路是使用其他生成式模型对他们进行评估。langchain也提供了一些试验性的接口来帮助实现这一行为

接下来将对上述六个部分的组件进行进一步的细致介绍

## Model I/O

在langchain中，其提供了一组构建集，用来整合LLM：

- prompts：模版化、动态生成、输入管理
- 语言模型：基于公开接口调用语言模型
- 输出parsing：对输出的结果进行解析

![](https://python.langchain.com/assets/images/model_io-1f23a36233d7731e93576d6885da2750.jpg)

在prompts环节，langchain提供两种方式：
1. prompts模版：对模型的输入进行参数化

    需要注意的是，prompts不能简单理解为chat中的输入文本，而应该是指输入的一组数据，这些数据和角色（role）进行了绑定，例如在openai的[chat completion](https://platform.openai.com/docs/guides/chat/introduction)中，角色就包括了AI, human以及system，chat的输出将会更遵循system的prompt input

2. example选择器：在prompts中动态选择输入样例

### prompts

feature store是传统机器学习中的概念，其保证喂给模型的数据是最新的和相关的[2]。

![](https://www.tecton.ai/wp-content/uploads/2020/10/feature-store-tecton-blog-preview.png)

## 参考

[1] https://github.com/hwchase17/langchain

[2] 