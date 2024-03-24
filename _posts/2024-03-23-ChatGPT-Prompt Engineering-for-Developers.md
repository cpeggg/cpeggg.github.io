---
title: ChatGPT Prompt Engineering for Developers
tags: AI
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->

[提示词工程](https://learn.deeplearning.ai/courses/chatgpt-prompt-eng/)学习笔记

<!--more-->

# 1. Intro
两种LLM：
- Base LLM：基于训练数据，根据prompt输出接下来的文字（类似completion）
- Instruction Tuned LLM：尝试根据指令做出对应的动作，除了基本的训练数据外，还需要提供针对instuctions和针对这些instuctions作出的good attempt的数据进行fine-tune，并结合一定的RLHF（reinforcement learning with human feedback）技术。

因此，instruction tuned LLM一般基于base LLM进行fine-tune得来。同时fine-tune的数据也可以让他变得更加helpful、honest和harmless

直观的回答对比如下：
![alt text](/assets/images/mdimages/20240323/image.png)

左边为base LLM，问特定问题的时候，它会尝试补全而不是回答问题；只有当我们给定特定的finetune训练集数据后，才会尝试进行右图中的回答。

出于以上的区别，推荐在生产应用中使用instuction tuned LLM，后面也基本上不会使用base LLM。而提示词工程所希望强调的，是在生产应用中，如何提供clear以及specific的提示词，来为实际的应用场景提供最为贴切的场景上下文描述。

# 2. GuideLines

prompting的两个原则：
1. write clear & specific instructions(clear != short)
2. give the model time to think

## 编写清晰指定的prompt

### 使用分隔符(delimiters)
一种常用的方式是使用分隔符（delimeters），例如：
```text
Summarize the text delimited by triple backticks into a single sentence.
```{text}```
```
它可以帮助做如下两件事：
1. clearly indicate the distinct part of the input
2. partially avoid prompt injection

![alt text](/assets/images/mdimages/20240323/image-1.png)

### 结构化输出(structured output)
另一种编写清晰特定的指令方法是要求使用结构化输出，如要求输出json/html格式输出，并在其中指定包含特定字段等信息

### 要求模型进行自检查（condition check）
看上去有点抽象，模型不是本来就按照prompt进行答复吗，这个的意义在哪？

举个例子，有如下的prompt模版：
```
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, then simply write "No steps provided."

"""{text_1}"""
```

这里的`If the text does not contain a sequence of instructions, then simply write "No steps provided."`就是条件检查。试想如果给定的text_1内容确实是一个指令序列，那么模型将总结并按上述的格式进行输入；但如果text_1实际内容并不是一个指令序列的表述，那么强行对其总结可能会带来非预期的结果，因此我们需要要求其进行检查并在不满足时输出特定的内容。

有条件检查和没有的区别如下：
![alt text](/assets/images/mdimages/20240323/image-2.png)
![alt text](/assets/images/mdimages/20240323/image-3.png)

### 给定一些预设的回答作为上下文(few-shot prompting)

在prompt模版中，先提供一些对话的回答示例，帮助LLM更好理解如何做出回答。这在缺乏上下文的（如特定语境下的对话、特定人物语气对话）prompt中尤为有用

## 给模型时间进行思考
模型和人一样，有的时候并不知道在特定情形下如何给出想要的回答，因此可以在prompt中对其进行引导，帮助其给出如何完成任务的思考，并最终帮助其给出想要的回答
### 指定完成任务的特定步骤描述
即。将任务拆解成特定的几个步骤描述，并要求模型按步骤输出结果（及相应的格式），帮助其得到指定的想要结果。

同时，对于让模型直接总结得到结论/判断某个论述的观点是否正确，一种办法是先让模型自己按解法思路来思考，在得到答案之后，再判断prompt中的input是否正确，例如如下的例子：
![alt text](/assets/images/mdimages/20240323/image-4.png)
这里由于LLM实际上和大多数人快速看一样，都只会对这里的学生解法进行略读，而不会仔细检查其中的Maintenance cost中实际为10x而非100x，而如果我们换成如下的prompt，让LLM判断学生的做法是否正确：
![alt text](/assets/images/mdimages/20240323/image-5.png)
可以发现，让LLM先自己实际去做，再去判断学生的做法是否正确，就可以得到正确的结论了。

## 模型的局限性

为什么模型会有幻觉：LLM并不能对自己是否知道某个事实有很清晰的边界把握，导致它对于实际它不知道的知识也会强行努力生成看似合理但实际上可能错误的答案。当前的算法研究正在尝试解决这一问题。

一种减轻幻觉的方法，是让LLM主动提供其总结经验的实际来源；比如对于一段话回答其中的特定问题，先要求其找出相关的文本引用，再在该基础上总结回答，可以帮助其降低幻觉产生的几率。

# 3. Iterative Prompt Development

在使用LLM构建实际应用时，通常很难通过一个prompt满足要求，而这也是为什么要使用交互式的方式来进行prompt开发：和代码开发类似，同样需要经历idea->impl->eval->feedback这样的loop来不断的完善我们的prompt设计。

而在prompt的开发过程中，你可能会遇到如下一些场景中特定存在的一些问题：
1. 返回的内容格式不符合要求
2. 返回的内容忽略了因上下文缺失而导致的细节缺失：比如对于关注商品型号的特定客户，在总结商品信息时，要求LLM提供型号的具体细节信息，而非单纯总结商品的features。

# 4. Summarizing

在进行总结文章类的应用开发时，有如下一些技巧：
1. 强调总结的字数限制
2. 强调侧重点
3. 让LLM摘抄（extract）而非总结（summarize）来避免幻觉

# 5. Inferring

推测，在LLM中通常指其有如下能力：
1. 判断观点（sentiment，negative/positive）
2. 判断情感（emotions）
3. 摘要，关键字，主题

# 6. Transforming

改变语言的形态通常指LLM有如下的能力：
1. 翻译：LLM通常由不同的语言输入训练而成，使其有翻译的能力
2. 语气转换：特定语气（如官话、俚语等），from casual to formal
3. 格式转换：比如将json转为html
4. 语法检查

# 7. Expanding

扩写，通常指LLM可以根据input、设定的语气、上下文，对input做出回复、扩写，其中的temperature参数，表示expanding中的的随机性大小，temperature值越大，生成的答案variety性更高：

![alt text](/assets/images/mdimages/20240323/image-7.png)

# 8. Chatbot

现在让我们回到completion的调用上来看，可以看到LLM的调用中，message字段长如下这样：

![alt text](/assets/images/mdimages/20240323/image-6.png)

之所以会有一个role：user，而且通常prompt是放到user对应的content下，是因为LLM在训练时，有user和assistant两个角色，user用来提供一系列的input信息，而assistant用来提供一系列的output信息用来返回：

![alt text](/assets/images/mdimages/20240323/image-8.png)

而实际上，可以有如下几种角色：
1. system: gives overall instructions（like 'you are a assitant...')
2. user: prompt main input(like 'tell me a joke')
3. assitant: return the output(like 'let me tell you a joke')

system role通常作为背景框架，可以不用成为实际对话上下文中的一部分，因此也通常不会被user角色所获取到原始要求。同时，LLM生成对话后，也会以assitant作为角色进行返回，因此：

```
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},    
{'role':'user', 'content':'Hi, my name is Isa'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```
的message，LLM会返回如下的结果：
```
{
  "content": "Hello Isa! It's nice to meet you. How are you doing today?",
  "role": "assistant"
}
```
而如果将上面的user直接改成assistant：
```
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},    
{'role':'assistant', 'content':'Hi, my name is Isa'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```
LLM会认为user没有说任何内容，因而会返回如下结果：
```
{
  "content": "How can I assist you today?",
  "role": "assistant"
}
```

而模型的对话上下文，实际上就是将每个role的每个content，按对话顺序输入到message，并将最新的input作为user append到messages队列中给到LLM，让其知道对话的上下文。

另外，system role也并非只能放在messages的最开头，你可以在messages中任意加入system role的input，实现一些管理级别的效果，like：

![alt text](/assets/images/mdimages/20240323/image-10.png)

# 9. Conclusion

![alt text](/assets/images/mdimages/20240323/image-9.png)

furthermore: 
[GPT-4 Techinical Report](https://cdn.openai.com/papers/gpt-4.pdf)
[Qwen Techinical Report](https://arxiv.org/abs/2309.16609)