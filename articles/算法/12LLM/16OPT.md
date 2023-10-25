论文：OPT: Open Pre-trained Transformer Language Models

机构：Meta AI

时间：2022.05.02

arxiv：[[2205.01068\] OPT: Open Pre-trained Transformer Language Models (arxiv.org)](https://arxiv.org/abs/2205.01068)

模型大小：125 million ~ 175 billion 的参数两

训练效果：OPT-175B 和 GPT-3 是 comparable 的，只用了 GPT3 的 1/7 的计算资源

计算资源：OPT-175B 使用了 992 张 80GB 的 A100 GPU，达到了每个 GPU 147 TFLOP/s

开源内容：代码、权重、训练笔记

# 背景

谁让GPT3不开源

## 训练方法

- 模型主要参考 《Language Models are Few Shot Learners》文章

- 权重初始化和 Megatron-LM 开源的代码中保持一致，使用了均值 0 标准差 0.006 的分布来初始化，输出层的标准差用 1.0/开根号(2L) 来做缩放，L是总共的层数

- 所有的偏置都是 0

- 整个模型的激活函数都是用 ReLU

- 序列的长度使用的是 2048

- 使用 AdamW 作为优化器，(β1,β2) = (0.9,0.95)，weight decay 是 0.1

- linear LR schedule

- - OPT-175B：在 2000 step 内，从 0 warm up 到最大的学习率
  - smaller baselines：在 375M token 内做 warm up，在 300B token 之后衰减 10% 的学习率

- mid-flight changes to LR：

- batch-size 从 0.5M 到 4M 变化，取决于模型的大小，并且在整个训练过程中保持不变

- dropout 始终使用 0.1，但是 embedding上不使用任何 dropout

- clip gradient norm 为 1，但是在 mid-flight changes 时会降低为 0.3

## 数据

预训练的数据语料包括（主要是英文文本，和少部分非英文文本）：

- RoBERTa：包括 BookCorpus、Stories、CCNews
- The Pile：包括 CommonCrawl, DM Mathematics, Project Gutenberg, HackerNews, OpenSubtitles, OpenWebText2, USPTO and Wikipedia. 所有子集都做了 ad-hoc whitespace 的 normalization.
- PushShift.ioReddit

数据集中 Jaccard similarity 大于 .95 的重复文档会被过滤掉，The Pile 数据集中重复文档很多。

Tokenize 使用的是 GPT-2 byte level BPE tokenizer

最后的语料中包括 180B 的 token

## 训练效率

在 992 台 80GB 的 A100 GPU 上训练 OPT-175B

使用了 Fully Sharded Data Parallel

使用了 Tensor Parallelism

达到了 147 TFLOPS

Adam 使用了 FP32，在所有 host 之间共享

模型权重保持 FP16

使用了动态损失缩放，避免 underflow

## 训练过程

下面是 OPT-175B 预训练过程中出现的重大训练调整：

- 硬件故障：35次手动重启，100台机器的循环（循环估计指的是除了故障，然后移出去）

- Loss Divergences：出现 loss diverge 的时候，降低学习率，从上一个健康的checkpoint重启训练可以解决问题

- - 健康的ckpt：dynamic loss scalar ≥ 1.0
  - 训练早期的时候，更低的gradient clipping(1.0修改到0.3)可以帮助稳定训练

- 其他 mid-flight 改变：作者使用了一系列经验性的方法来处理 loss divergences：

- - 换成vanilla SGD（优化很快就停滞了，又换回了AdamW）
  - 重置动态损失标量（有一些帮助，但不能处理所有情况）
  - 更换成更新版本的megatron（减少了activation norms 的压力，提高了吞吐量）

