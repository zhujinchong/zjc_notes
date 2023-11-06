论文：Training language models to follow instructions with human feedback

时间：2022.05.04

机构：OpenAI

arxiv: [[2203.02155\] Training language models to follow instructions with human feedback (arxiv.org)](https://arxiv.org/abs/2203.02155)

# Abstract

把语言模型变大并不意味着会让模型更好的理解用户意图，例如大的语言模型会生成一些不真实、有害的、没有帮助的输出给用户，换句话说，这些模型并没有和用户的意图对齐（aligned）。在这篇论文中我们展示了通过使用用户反馈来微调模型的方法，从而使得语言模型在一系列任务上与用户意图对齐。首先通过人工撰写和OpenAI API两种方式收集问题(prompts)，然后人工来写这些问题的答案，从而构建成一个数据集，再使用这些数据集对GPT3进行有监督的微调；我们又通过对模型的输出进行(人工)排序构建一个数据集，在这个数据集上，我们从人类反馈中通过强化学习进一步微调这个有监督模型，我们把最终得到的模型称之为InstructGPT。尽管InstructGPT的参数是GPT3的1/100，但是在我们（收集的）问题上通过人类评估发现，只有1.3B参数的instructGPT模型输出比有175B参数的GPT3模型输出更好，此外InstructGPT在真实性和降低有害输出上都有所改善，并且在公开NLP数据集上的表现也不逊色。虽然InstructGPT仍然会犯简单的错误，但是我们的结果证明，使用人类反馈来微调语言模型从而使得模型与人类意图对齐是一个正确的方向。

# 1 Introduction

通过给定（当前）任务的一些样本作为输入，大语言模型可以通过prompted的方式去执行一系列NLP任务，然而这些模型经常表现出意料之外的结果，例如捏造事实，生成有偏见或有害的内容，或者完全不服从用户的指令。这是因为当前很多大模型的训练目标是在来自网络上爬取的数据集上预测下一个字，这完全与“根据用户指令生成有帮助的安全的内容”的目标不符。因此我们认为语言模型的目标并没有对齐(misaligned)。对于那些被部署在成千上万的应用上的语言模型，避免这种意料之外的行为尤为重要。我们最终目的希望语言模型是有帮助的（语言模型应该帮助用户解决任务），真诚的（语言模型不应该编造信息、误导用户）、无害的（语言模型不应该对人或社会造成生理上、心理上或社会危害）

文章专注于通过微调的方式去对齐语言模型，具体来说，就是从人类反馈中使用强化学习去微调GPT3，使其遵循一系列的手写指令。这个技术使用人类偏好作为奖励信号去微调我们的模型。具体方法如下图所示
（1）首先基于外包人员的测试表现，选择了40个外包去标注数据
（2）基于人openAI API和人工两种方式获取一些问题，让外包写这些问题的期望答案，从而构建成一个数据集，然后使用这个数据集去有监督的训练GPT3作为baseline
（3）通过API获取更多的问题集，让人工对模型对这些问题的输出进行排序，从而又构建了一个数据集B
（4）在数据集B上训练一个奖励模型(RM)去预测哪个模型的输出是更符合人类期望的
（5）使用奖励模型作为奖励函数去微调baseline模型，使用PPO算法最大化奖励
（6）基于GPT-3模型架构，通过上述方式训练得到的模型就是InstructGPT

图2：InstructGPT方法

![image-20230806111140861](images/image-20230806111140861.png)

我们主要通过让我们的劳工评价我们的测试集中的模型输出的质量来评估我们的模型，包括来自保留的客户（他们在培训数据中没有代表）的提示。我们还对一系列公共的NLP数据集进行了自动评估。我们训练三种模型大小 (1.3B, 6B, and 175B parameters）。我们所有的模型都使用GPT-3体系结构。我们的主要发现如下：

（1）外包都认为InstructGPT输出结果比GPT-3好：1.3B的InstructGPT效果要比175B的GPT3的结果好
（2）相比GPT-3，InstructGPT在真实性上表现更好
（3）相比GPT-3，InstructGPT在减少生成有害信息上效果更好，但是对于偏见方面，并无啥提升
（4）在对话生成上InstructGPT有明显提升，在NLP公开数据集上表现退化很小
（5）即使没有参与到标注训练数据的外包也认为InstructGPT生成的结果要比GPT3结果好（每个人对问题的答案好坏有主观性，参与到训练集标注的外包可能在某些方面有“偏见”，为了减少这种偏见，因此初步有请了没有参与到训练集标注的外包来测试结果）
（6）微调对数据比较敏感：在人工标注数据上微调GPT3 vs 在公开数据集上微调GPT3，在来自API的问题集上测试，表现还是前者更强
（7）模型有一定的泛化性，即使针对很少出现在微调数据集中的instruction也可以回答很好
（8）InstructGPT仍然会范一些简单的错误

# 2 Related work

# 3 Methods and experimental details

## 3.1 High-level methodology

模型训练方法：

步骤1：收集示范数据，并培训一个有监督的策略。我们的标签符在输入提示分布上提供了所需行为的演示（有关此分布的详细信息，请参见第3.2节）。然后，我们使用监督学习方法对该数据进行了预先训练过的GPT-3模型的微调。

第二步：收集比较数据，并训练一个奖励模型。我们收集模型输出之间的比较数据集，其中标记符表示它们更喜欢给定输入的哪个输出。然后，我们训练一个奖励模型来预测人类偏好的输出。

步骤3：使用PPO针对奖励模型优化策略。我们使用RM的输出作为标量奖励。我们使用PPO算法对监督策略进行微调，以优化该奖励（Schulman et al.，2017）。

步骤2和步骤3可以连续迭代；根据当前的最佳策略收集了更多的比较数据，用于训练一个新的RM，然后再训练一个新的策略。实际上，我们的大部分比较数据来自我们的监督政策，其中一些来自我们的PPO政策。

## 3.2 Dataset

1）通过如下三种方式让外包写promts（这些promts一般不会出现在GPT3 API中），来训练初版的InstructGPT

* Plain: 仅仅让外包想任意的任务，且这些任务具有足够的多样性
* Few-shot：让外包想一些instruction以及针对这个指令的多个query/response pair‘
* User-based：让外包根据openAI API waitlist中的用例想出相对应的问题

2）基于以上初版InstructGPT制作了一个Playground让用户使用，因此训练Instruct GPT的prompt数据集主要来自基于初版InstructGPT的openAI API

基于以上prompts制作了三个不同的数据集用于微调过程：1）SFT数据集 13K --> 训练SFT模型 2）奖励模型数据集 33K --> 训练RM　3) PPO数据集（无任何人工label）31k --> RLHF训练

总结来说数据集构建的过程为：人工写一些prompts–>构建初版InstructGPT – > 从初版InstructGPT API中选择更多的 prompts --> 人工写prompts的答案及排序 – > RLHF训练

## 3.3 Tasks

我们的训练任务来自两个来源： 

(1)由我们的标签人员编写的提示数据集

(2)在我们的API上提交给早期InstructGPT 模型的提示数据集（见表6）。

这些提示非常多样化，包括生成、问题回答、对话、总结、提取和其他自然语言任务（见表1）。我们的数据集是超过96%的英语数据集，但是在第4.3节中，我们也探讨了我们的模型对其他语言中的指令的响应和完成编码任务的能力。

对于每个自然语言提示，任务通常是通过自然语言指令直接指定的（例如，“写一个关于聪明的青蛙的故事”），但也可以通过一些镜头的例子（例如给两个青蛙故事的例子，并促使模型生成一个新的例子）或隐式延续（例如提供一个关于青蛙故事的故事的开始）。在每种情况下，我们都要求我们的标签人员尽最大努力来推断编写提示符的用户的意图，并要求他们跳过任务非常不清楚的输入。此外，我们的劳工还考虑了隐含的意图，如反应的真实性，以及潜在的有害输出，如有偏见或有毒的语言，根据我们提供的指导（见附录B）和他们的最佳判断。

## 3.5 Models

# 4 Results

# 5 Discussion



