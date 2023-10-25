参考：

Shaw在2018的提出的相对位置编码  https://www.cnblogs.com/shiyublog/p/11185625.html 

Transformer-XL在2019年中用到的相对位置编码 https://www.cnblogs.com/shiyublog/p/11236212.html 



2019/1/9，《Transformer-XL: Attentive Language Models Beyond a Fixed-Length Context》

 https://arxiv.xilesou.top/pdf/1901.02860.pdf 

Transformer是一种潜力巨大的特征提取器或者基础编码器，但在**语言建模的设置中受到固定长度上下文的限制**。 

Transformer XL是Transformer的第三代产品，它的前两代分别为：

* 第一代， 《Attention Is All You Need》 
* 第二代， 《Universal Transformer》，更具有通用性，新的并行计算结构。

Transformer XL的两个创新点:

* 片段级递归机制(segment-level recurrence mechanism)，引入一个记忆模块，循环用来建模片段间的联系。解决了上下文碎片化问题
* 相对位置编码机制(relative position embedding scheme)，代替绝对位置编码，避免时序混淆。在shaw 2018论文提出的相对位置编码之上的改进。

### 第二代结构Vanilla Transformer

![img](images/v2-8ddfd89bcc6efb3fa54727ed937883c1_hd.jpg)

在训练时，将整个语料库分割成512的片段，在每个片段中训练模型，忽略来自前一段的所有上下文信息；在评估阶段，传统的Transformer模型在每个步骤都消耗与训练期间相同长度的一个segment。然后，在下一步中，这个segment向右移动一个位置，并从头开始处理，只在最后一个位置只是进行一次预测。 

缺点：

* 上下文长度受限：字符之间的最大依赖距离受输入长度的限制，模型看不到出现在几个句子之前的单词 。
* 上下文碎片化： 对于长度超过512个字符的文本，都是从头开始单独训练的。段与段之间没有上下文依赖性，会让训练效率低下，也会影响模型的性能。 
* 推理速度慢： 在测试阶段，每次预测下一个单词，都需要重新构建一遍上下文，并从头开始计算，这样的计算速度非常慢。 

### Transformer XL（extra long）

![img](images/v2-a8650a299d029389959a23d4fb3ef32b_hd.jpg)

在训练过程中，对上一个segment计算的隐藏状态序列进行固定和缓存，并在模型处理下一个新的segment时对其进行利用。在评估阶段，可以重用前面部分的表示，而不是像传统模型那样从头开始计算，这样可以提高速度。 ( 在评估阶段，与vanilla Transformer相比，其速度也会更快。在vanilla Transformer中，一次只能前进一个step，并且需要重新构建段，并全部从头开始计算；而在Transformer-XL中，每次可以前进一整个段，并利用之前段的数据来预测当前段的输出。 )



### 片段递归机制

**计算当前序列当前层的隐藏状态时，引入前一个序列上一层的隐藏状态**。  TransformerXL的做法很简单，就是按照序列长度的维度将他们concate起来。 如下的公式所示：  

![img](images/v2-a4571af0e286733c0ea454c4d99769de_hd.jpg)

其中h代表Transformer在第r个segment上，第n隐层的输出。第一个公式中，SG()表示梯度停止，也就是不做更新，[]表示两个向量的拼接。

从公式可以看出，模型在当前segment中加入了前一个segment的浅隐层输出，然后将这些信息加入到Key和Value向量中，这样就在特征提取时就用到了上文的信息，从图中可以看出，最长的依赖长度随着隐层数和 segment长度呈线性增长，也就是O（N×L），这样就大大增加了依赖长度，解决了context fragmentation。 



### 相对位置编码

TransformerXL引入了一种Relative Positional Encodings机制，会根据词之间的相对距离而非像传统的Transformer中的绝对位置（正余弦）进行编码。 

传统Transformer中，计算qi和kj之间的attention分数如下：E_xi是词xi的embedding，E_xj是xj的embedding, Ui 和Uj 是绝对位置向量 。query[E,U]xkey[E,U]

（ 应用乘法分配率， query的embedding 分别与 key的embedding, positional encoding相乘相加；之后 query的positional encoding分别与 key的embedding, positional encoding相乘相加 ）

![img](images/v2-5b778b05733b4db5a674b8c5ae0a6c2c_hd.jpg)

i表示行，j表示列，x是矩阵

* a项没有包含位置信息，表示代表i行的字对j列的字提供多大的注意力。
* b项捕获的是在row i的字对其他位置的关注信息。
* c项捕获的是在col j的字对其他位置的关注信息。
* d项捕获的是模型的global attention，表示position i对position j付出多大的注意力，例如两个字的位置越远，期望它们之间的注意力越小。

在Transformer-XL中，对上述的attention计算方式进行了变换，转为相对位置的计算，而且不仅仅在第一层这么计算，在每一层都是这样计算：

![img](images/v2-c317c5fbfeaf38c434778a4d9593259f_hd.jpg)

主要有三点变化：

1）在b和d这两项中，将所有绝对位置向量**Ui**，**Uj**都转为相对位置向量**Ri−j**，与Transformer一样，这是一个固定的编码向量，不需要学习。

2）在c这一项中，将查询的**U_i**^T***W_q**^T向量转为一个需要学习的参数向量**u**。 **因为无论query在序列中的绝对位置如何，其相对于自身的相对位置都是一样的**。这说明与绝对位置无关，应当保持不变，所以用一个可学习参数代替。 

3）将**K**的权重变换矩阵**Wk**转为**Wk_E** 和**Wk_R**，分别作为content-based key vectors和location-based key vectors。而原来是采用同样的相信变换，因为位置编码固定。

经过变换后，每一项都有了一个直观上的表征含义：

a.表示基于内容的表征

b.表示基于内容的位置偏置

c.表示全局的内容偏置

d.表示全局的位置偏置

总的来说，**Relative Positional Encodings就是在计算attention分数时，用相对位置R_i_j编码来代替原来的绝对位置编码Ui和Uj。并且学习了相对位置v和u用来调整不同距离和不同嵌入的得分。**

### 计算

绝对位置编码计算如下：z = softmax(qk)v

![img](images/1453927-20190714210621814-1849661371.png)

相对位置编码计算如下：

![img](images/1453927-20190726123759232-1609808719.png)

### 高效计算

在计算aij时，时间复杂度为O(n^2)，论文中提出了O(n)的方法。



 ### 总结

总的来说TransformerXL对Transformer进行了一些调整，试图解决一些问题。按照论文的描述，**TransformerXL学习的依赖关系比RNN长80%，比传统Transformer长450%，在短序列和长序列上都获得了更好的性能，并且在评估阶段比传统Transformer快1800+倍**。 



### 相对位置编码



