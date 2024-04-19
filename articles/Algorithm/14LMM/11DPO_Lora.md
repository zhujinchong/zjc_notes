[DPO——RLHF 的替代之《Direct Preference Optimization: Your Language Model is Secretly a Reward Model》论文阅读 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/634705904)

[

UCLA华人提出全新自我对弈机制，LLM自己训自己，效果碾压GPT-4专家指导-36氪 (36kr.com)](https://www.36kr.com/p/2632885145549960)





[OpenAI：Superalignment的一种途径——Weak-to-Strong Generalization - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/685280689)





# DoRA

https://zhuanlan.zhihu.com/p/682556551



为了充分释放预训练模型在聊天场景中的潜力，Meta 还对指令微调方法进行了创新。Llama 3 后训练方法用的是有监督微调（SFT）、拒绝采样、近端策略优化（PPO）和直接策略优化（DPO）的组合。SFT 中使用的 prompt 质量以及 PPO 和 DPO 中使用的偏好排序对模型对齐有着巨大的影响。

此次模型质量的最大改进，来自于仔细整理数据以及对人类注释者提供的注释进行多轮质量保证。

通过 PPO 和 DPO 从偏好排序中学习，也极大地提高了 Llama 3 在推理和编码任务上的性能。Meta 发现，如果你向模型提出一个它难以回答的推理问题，该模型有时会产生正确的推理轨迹：模型知道如何产生正确的答案，但不知道如何选择它。对偏好排序的训练使模型能够学习如何选择正确答案。