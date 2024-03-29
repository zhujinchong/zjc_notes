# Mamba

UCLA华人提出全新自我对弈机制！LLM自己训自己，效果碾压GPT-4专家指导 (qq.com)](https://mp.weixin.qq.com/s/ORZCxa97y-Hn8yme97eBTA)





[微软这篇论文把LLM的上下文扩展到了200万tokens (qq.com)](https://mp.weixin.qq.com/s/9mcccgEICn-jU6Xl4vFltg)



[一文通透想颠覆Transformer的Mamba：从SSM、HiPPO、S4到Mamba_mamba模型-CSDN博客](https://blog.csdn.net/v_JULY_v/article/details/134923301)



Mamba: Linear-Time Sequence Modeling with Selective State Spaces

时间：2023-12



背景:

为了解决 Transformer 在长序列上的计算效率低下问题，人们开发了许多亚二次时间架构，如线性注意力、门控卷积和递归模型以及结构化状态空间模型（SSM）。此类模型的一个关键弱点是**无法进行基于内容的推理**。

其中SSMs最有潜力，它可以被解释为**递归神经网络（RNN）和卷积神经网络（CNN）的结合**，其灵感来自经典的状态空间模型（卡尔曼，1960）。但它们在离散和信息密集数据（如文本）的建模方面不太有效。





# Jamba



https://mp.weixin.qq.com/s/lCUr6qN1SkyRJTZWI9Jtvg