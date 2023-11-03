论文：GPT-NeoX-20B: An Open-Source Autoregressive Language Model

机构：EleutherAI 

时间：2022.04.14

arxiv: [[2204.06745\] GPT-NeoX-20B: An Open-Source Autoregressive Language Model (arxiv.org)](https://arxiv.org/abs/2204.06745)

## 摘要

我们介绍了 GPT-NeoX-20B，这是⼀种在 Pile 上训练 的200 亿参数⾃回归语⾔

## 1.介绍

在过去⼏年中，围绕⽤于⾃然语⾔处理的⼤语⾔模型 (LLM) 的研究呈爆炸式增⻓，这在很⼤程度上归功于BERT等基于Transformer的语⾔模型令⼈印象深刻的性能，在此基础上产生一系列的模型，如GPT-2、 GPT-3和T5。这项研究最有影响⼒的成果之⼀是发现， LLM的性能随着参数数量增加而增加。

目前，存在数⼗个公开的LLM。最⼤的模型的参数⽐GPT-2多两个数量级，在这个层级上有⼗⼏种不同的模型。然⽽，这些模型⼏乎普遍是⼤型科技公司的受保护知识产权。

作者通过调查发现参数数量大于GPT2的LLM有：
GPT-Neo、 GPT-J-6B 、 Megatron-11B1 、Panguα-13B和FairSeq

在本⽂中，我们介绍了GPT-NeoX-20B，这是⼀个 200亿参数的开源⾃回归语⾔模型。我们通过许可⾃由和公开地向公众提供模型权重，其动机是相信LLM 的开放访问对于推进⼴泛领域的研究,特别是在AI 安全、可解释性。 LLM的许多最有趣的功能只出现在⼀定数量的参数之上，并且它们具有许多⽆法在较⼩模型中研究。

## 2.模型设计和实现

GPT-NeoX-20B是一种自回归Transformer解码器模型，其架构在很大程度上遵循GPT-3的架构，但存在如下所述的一些显着偏差。我们的模型有 200 亿个参数，其中199亿个是Kaplan等人提出的“非嵌入”参数。我们的模型有 44 层，隐藏维度大小为6144，有64个头。