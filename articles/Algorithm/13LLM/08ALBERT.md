参考：

* https://zhuanlan.zhihu.com/p/84273154



2019/9 ALBERT (from Google)

ALBERT: A Lite BERT for Self-supervised Learning of Language Representations

https://arxiv.org/abs/1909.11942v1



解决问题：

1. 让模型参数更少
2. 使用更少的内存
3. 提升模型效果



现象：

当模型参数变多，一开始模型效果提升，但提升到一定程度，效果会降低，这个现象叫做"model degratation"

ALBERT提出了三种优化策略：

1. 压缩词向量
2. 共享层级之间的参数
3. 修改next sentence损失函数



### 提出的三个改进：

**1. Factorized Embedding Parameterization**

在BERT、XLNet、RoBERTa中，词表的embedding size(E)和transformer层的hidden size(H)都是相等的，这个选择有两方面缺点：

1. 从建模角度来讲，wordpiece向量应该是不依赖于当前内容的(context-independent)，而transformer所学习到的表示应该是依赖内容的。所以把E和H分开可以更高效地利用参数，因为理论上存储了context信息的**H要远大于E**。
2. 从实践角度来讲，NLP任务中的vocab size本来就很大，如果E=H的话，模型参数量就容易很大，而且embedding在实际的训练中更新地也比较稀疏。

因此作者使用了小一些的E(64、128、256、768)，训练一个独立于上下文的embedding(VxE)，之后计算时再投影到隐层的空间(乘上一个ExH的矩阵)，相当于做了一个因式分解。

从后续的实验中来看，E的大小与实验效果也不是完全正相关，因此其他实验中E都取的128。



**2. Cross-layer parameter sharing**

跨层参数共享，就是不管12层还是24层都只用一个transformer，然后循环。（这个观点来自论文：Universal Transformers）

![image-20230320232050611](images/v2-c4d5761a83544bfcc5eaffb266d92e8d_hd.jpg)



**3. Inter-sentence coherence loss**

BERT有两个目标： masked language modeling(MLM) and   next-sentence prediction(NSP)。

后BERT时代很多研究(XLNet、RoBERTa)都发现NSP没什么用处，所以作者也审视了一下这个问题，认为NSP之所以没用是因为这个任务不仅包含了句间关系预测，也包含了主题预测，而主题预测显然更简单些（比如一句话来自新闻财经，一句话来自文学小说），模型会倾向于通过主题的关联去预测。因此换成了SOP(sentence order prediction)，预测两句话有没有被交换过顺序。实验显示新增的任务有1个点的提升。



### 总结

刚开始看这篇文章是很惊喜的，因为它直接把同等量级的BERT缩小了10倍+，让普通用户有了运行GPT2、威震天的可能。但是仔细看了实验后才发现体量的减小是需要付出代价的：

![img](images/v2-a54a79b04d564565974ae68641c0317a_hd.jpg)

实验使用的xlarge是24层，2048维度，xxlarge是12层，4096维度。可以仔细看一下model的量级，并且注意一下这个speedup是训练时间而不是inference时间（因为数据少了，分布式训练时吞吐上去了，所以ALBERT训练更快），但inference还是需要和BERT一样的transformer计算。

