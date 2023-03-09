---
title: "10行代码调用ChatGPT"
date: 2023-03-09T21:55:02+08:00
tags: [ChatGPT, OpenAI]
categories: [OpenAI]
---

OpenAI无疑是人工智能应用领域的领先者之一。该组织致力于研究人工智能的前沿技术，以及探索如何将这些技术应用于实际问题。它的研究和开发项目覆盖了广泛的领域，包括自动化、机器人、语音识别和自然语言处理等。

<!-- more -->

下面是OpenAI api首页列出的一些功能以及使用指南：

![图1 OpenAI API首页](/img/AI/OpenAI/openai-api-index.png)

图1 OpenAI API首页

我们可以看到有`Chat` ，有文本生成，有基于向量的搜索、分类以及文本对比，有语音转文本，有图像生成，有代码生成，还有大模型的精调。其实还有很多很好玩的功能没有列出来，以后可以再写文章介绍更多的玩法。

这次咱们就来演示如何调用 `Chat` 的api实现类似ChatGPT的魔法。

OpenAI不仅宣布开放，价格还直接打了个骨折：0.002美元/每1000 token，仅为此前GPT-3.5（text-devinci-003）价格的1/10。ChatGPT API 质优价廉，开发者胖友们可以赶快用起来了。

下面就是示例代码，10行代码实现ChatGPT：

```python
import openai
openai.api_key = "API_KEY"
def ask_chatgpt(text):
    resp = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "user", "content": text}
        ],
        temperature=0，
    )
    return resp.choices[0].message
```

上面的11行Python代码实现了ChatGPT模型的调用，把temperature设置为0，可以让结果更加稳定，temperature最大值为1，值越大结果更不一样，如果想规避查重风险，直接把temperature参数值调到最大。

下面来对比一下gpt-3.5-turbo（ChatGPT/GPT-3.5）与text-devinci-003(GPT-3)的输出结果。以Java代码写快速排序为例：

![图2 gpt-3.5-turbo的输出结果](/img/AI/OpenAI/gpt-3.5-turbo-quick_sort.png)

图2 gpt-3.5-turbo的输出结果

![图2 text-devinci-003的输出结果](/img/AI/OpenAI/text-davinci-003-quick_sort.png)

图2 text-devinci-003的输出结果

可以很明显的看到gpt-3.5-turbo（ChatGPT/GPT-3.5）不仅输出了完整的代码（拷贝下来直接运行），还给出了代码的解释。

是不是更加香了，大家赶快用起来吧。