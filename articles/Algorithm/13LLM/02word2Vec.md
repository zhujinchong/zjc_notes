有两种方式实现word2Vec，一种是CBOW模型，从一个句子里面把一个词抠掉，用这个词的上文和下文去预测被抠掉的这个词；一种是skip-gram模型，输入某个单词，要求网络预测它的上下文单词。

![img](images/640.webp)



# 1. skip-gram

skip-gram要点：

1. 是一个三层的神经网络
2. 训练：输入是一个word，输出是word的周边的词
3. 然后去掉最后一层，只保存input_layer和hidden_layer
4. 预测：输入一个word，hidden_layer将会给出该词的embedding repesentation



### 第一步，获取训练数据

注意：如果这个词是一个句子的开头或结尾， 忽略窗外的词。

![img](images/640-1572829481674.webp)

接着，对上面的词进行word2int处理，并将其转成one-hot向量



### 第二步，训练

输入层到隐藏层：

![img](images/640-1572829706611.webp)

由上图，我们可以看出，我们将Input转换成embedding_representation，并将vocab_size维度降低到embedding_dim.

隐藏层到输出层，输出层使用softmax函数预测该word周边词的概率。

![img](images/640-1572829836100.webp)

所以整体的过程为：

![img](images/640-1572829856020.webp)



### 总结

![1572829940467](images/1572829940467.png)

![1572830136341](images/1572830136341.png)

注意：

seq2seq模型，输入处都会乘以embedding_matrix，输出都会乘以embedding_matrix^T，这两个embedding矩阵有时会共享，又是则不会。

