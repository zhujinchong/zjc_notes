## self-evoluation

我们在预训练数据中混合入10%的监督微调数据（a random sample of sft data），用非监督的格式。背后的直觉是预训练让模型专注基础知识p(t_n | t_n-1…)，监督学习让模型专注指令完成p(response | instruction)。在sft的训练过程中，大部分的gradient会指向完成指令的方向，因为数据中的基础知识已经在预训练中学习过。这就是我们让模型自我进化的思想（self-evoluation），这和人类循序渐进的学习知识一个原理；比如，NLP的学生总是先学好各种概率分布，然后再学习在各种任务数据中的应用，打好基础，事半功倍。

# 13B模型

8月21日

TigerBot-13B-base: 基于Llama-2-13B继续预训练300B tokens，扩充了中文词表到60K vocabulary, 并采用holistic training在预训练中直接使模型具有9成的指令完成能力。在主流英文基准测试中超过Llama-2-13B-base的7%，在中文测试中综合能力超过Llama-2-13B-base的49%，在国内主流开源基座模型中处于领先位置。

TigerBot-13B-chat: 基于TigerBot-13B-base用5M指令数据微调，并采用rejection sampling fine-tune对齐人类需求。在主流英文基准测试中达到Llama-2-13B-chat的 101%，在中文测试中综合能力超过Llama-2-13B-chat的47%，在国内主流开源模型中亦处于领先位置。

## 数据

为了补充Llama在中文上的不足，我们继续预训练了1TB数据，约300B tokens，中英文比例约1:1。其中去除了Llama预训练用到的数据(commoncrawl, c4)，补充了中文数据（书籍，百科，网页等）。

## 词表

为了弥补Llama中文数据的不足（约占其训练数据的0.13%），我们扩充了中文进入词表。首先我们从中文训练数据中随机抽取10%，约100G数据，训练了中文为主的sentencepiece tokenizer (感谢@google/sentencepiece)。因为中英文语言的自然分布不同，我们扩充的tokenizer以中文为主，保证中文在最后tokenizer中充分表现，然后合并到Llama-2的tokenizer中（感谢@ymcui/Chinese-LLaMA-Alpaca），最终tokenizer（tigerbot_llama.model）vocabulary size达到60,515；tokenizer效率（raw:tokenized）达到llama的约2倍。我们也发现tokenizer训练数据100G以上没有必要，覆盖的字符基本一致，并且cpu峰值可超过2TB。

## holistic training

我们在Tigerbot-v2的实验中提出self-evolution技术，在此基础上进一步提出holistic training的方法：将预训练中的指令完成类数据达到预训练数据的5-10%，这可以达到sft数据的5-20倍；并且在原预训练数据中去重掉和sft类数据知识重复的部分。背后的思想是：指令完成（问答、生成等）是人类语言续写的一部分，我们希望模型在学知识的同时也学习指令完成的模式。这样带来的好处是两方面的：

1. base model不经过sft就立刻展现出很强的指令完成能力，并且是更泛化的；
2. 由于基础的能力（知识和指令完成的格式）已经在pretrain中学得，sft可以很轻量，而把有限的计算资源投入到alignment中（如ppo, rejection-sampling）。我们实验发现，sft训练大约1万条数据后loss就基本达到convergence的95%

## rejection sampling

在sft训练中，我们加入了rejection-sampling fine-tune，算法如下:

1. 随机取样sft数据中5-10%的prompts，
2. 用best candidate chat model对每个prompt随机生成10个generation，
3. 其中95%用reward model ranking （smooth Pareto frontier），5%用human labeling ranking (push Pareto frontier)，1%用human labeling editing (jump Pareto frontier)，
4. 将top ranked generation混合入sft训练数据，然后训练。



# 70B模型

我们很高兴地发布Tigerbot-70b，继续开源和免费商用，包括：

1. Tigerbot-70b-base: 在Llama-2-70b的基础上继续预训练，模型综合能力在mmlu等10项主流基准测试中，优于Llama-2-70b，达到业内SOTA。

   a. 用高质量的300 billion tokens的多语言数据，

   b. 算法上使用了GQA, flash-attn, RoPE, holistic-training等技术，

   c. 训练采用了tensor/pipeline-partition技术，计算效率达到Llama-2 paper中报告的SOTA。

2. Tigerbot-70b-chat: 在Tigerbot-70b-base基础上，用20M指令完成数据进行sft，和10K人类标注的gold set进行rejection-sampling对齐。



