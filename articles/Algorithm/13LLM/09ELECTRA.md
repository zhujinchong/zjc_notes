参考：  https://zhuanlan.zhihu.com/p/89763176 

2019/9 斯坦福SALL实验室《 Efficiently Learning an Encoder that Classifies Token Replacements Accurately 》

创新点：

1. GAN架构
2. 仅用1/4算力达到了RoBERTa的效果

### 模型

ELECTRA最主要的贡献是提出了新的预训练任务和框架，把生成式的Masked language model(MLM)预训练任务改成了判别式的Replaced token detection(RTD)任务，判断当前token是否被语言模型替换过。那么问题来了，我随机替换一些输入中的字词，再让BERT去预测是否替换过可以吗？可以的，因为我就这么做过，但效果并不好，因为随机替换**太简单了**。 

于是作者就干脆使用一个MLM的G-BERT来对输入句子进行更改，然后丢给D-BERT去判断哪个字被改过，如下：

 ![img](images/v2-2d8c42a08e37369b3b504ee006d169a3_hd.jpg)