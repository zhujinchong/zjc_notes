2019.1.31 微软

Multi-Task Deep Neural Networks for Natural Language Understanding

https://arxiv.org/abs/1901.11504



思想：

两个阶段：预训练和多任务微调。预训练和BERT一样。微调阶段：将四个任务的数据打乱，随机拿出一个任务的batch训练，提高模型泛化能力。



四个独立任务：

- 单句分类：采用[CLS]作为句子编码表示，softmax损失函数
- 文本相似度：采用[CLS]作为句子编码表示，sigmoid损失函数
- 两个句子分类：分别将第一句和第二句的词编码拼接，参考[SAN模型](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1712.03556)迭代推理结果
- 相关性排序：采用[CLS]作为句子编码表示，sigmoid损失函数



