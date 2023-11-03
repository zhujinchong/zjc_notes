2019/7 RoBERTa

RoBERTa: A Robustly Optimized BERT Pretraining Approach

https://arxiv.org/abs/1907.11692



第三章：

1. Adam的参数由BERT的0.99调为0.98；
2. BERT使用16G数据；本文使用160G;
3. BERT会使用短文本训练；本文使用全长文本训练。

第四章：

1. BERT每个掩码训练四次，称为静态掩码；本文每个训练样本动态掩码;
2. 不适用NSP loss，使用全长句子512tokens，两个文档之间的首尾句子加separator token。
3. 过去研究表明提高学习率和batch_size会变好，BERT也同样，原来256batch_size，提高到2K
4. larger byte-level BPE编码

第五章：

* RoBERTa_larger使用160GB_data+500K_steps最终登顶（模仿的BERT_larger）。在后文中，三个benchmarks评估，都是用这个larger模型。

