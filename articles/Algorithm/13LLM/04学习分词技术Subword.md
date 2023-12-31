# NLP三大Subword模型详解：BPE、WordPiece、ULM

在NLP任务中，神经网络模型的训练和预测都需要借助词表来对句子进行表示。传统构造词表的方法，是先对各个句子进行分词，然后再统计并选出频数最高的前N个词组成词表。通常训练集中包含了大量的词汇，以英语为例，总的单词数量在17万到100万左右。出于计算效率的考虑，通常N的选取无法包含训练集中的所有词。因而，这种方法构造的词表存在着如下的问题：

- 实际应用中，模型预测的词汇是开放的，对于未在词表中出现的词(Out Of Vocabulary, OOV)，模型将无法处理及生成；
- 词表中的低频词/稀疏词在模型训练过程中无法得到充分训练，进而模型不能充分理解这些词的语义；
- 一个单词因为不同的形态会产生不同的词，如由"look"衍生出的"looks", "looking", "looked"，显然这些词具有相近的意思，但是在词表中这些词会被当作不同的词处理，一方面增加了训练冗余，另一方面也造成了大词汇量问题。

一种解决思路是使用字符粒度来表示词表，虽然能够解决OOV问题，但单词被拆分成字符后，一方面丢失了词的语义信息，另一方面，模型输入会变得很长，这使得模型的训练更加复杂难以收敛。

针对上述问题，Subword(子词)模型方法横空出世。它的划分粒度介于词与字符之间，比如可以将”looking”划分为”look”和”ing”两个子词，而划分出来的"look"，”ing”又能够用来构造其它词，如"look"和"ed"子词可组成单词"looked"，因而Subword方法能够大大降低词典的大小，同时对相近词能更好地处理。

目前有三种主流的Subword算法，它们分别是：Byte Pair Encoding (BPE), WordPiece和Unigram Language Model。

## Byte Pair Encoding (BPE)

BPE最早是一种数据压缩算法，由Sennrich等人于2015年引入到NLP领域并很快得到推广。该算法简单有效，因而目前它是最流行的方法。GPT-2和RoBERTa使用的Subword算法都是BPE。

BPE获得Subword的步骤如下：

1. 准备足够大的训练语料，并确定期望的Subword词表大小；
2. 将单词拆分为成最小单元。比如英文中26个字母加上各种符号，这些作为初始词表；
3. 在语料上统计单词内相邻单元对的频数，选取频数最高的单元对合并成新的Subword单元；
4. 重复第3步直到达到第1步设定的Subword词表大小或下一个最高频数为1。

实际上，随着合并的次数增加，词表大小通常先增加后减小。在得到Subword词表后，针对每一个单词，我们可以采用如下的方式来进行编码：


1. 将词典中的所有子词按照长度由大到小进行排序；
2. 对于单词w，依次遍历排好序的词典。查看当前子词是否是该单词的子字符串，如果是，则输出当前子词，并对剩余单词字符串继续匹配；
3. 如果遍历完字典后，仍然有子字符串没有匹配，则将剩余字符串替换为特殊符号输出，如`<unk>`；
4. 单词的表示即为上述所有输出子词。

解码过程比较简单，如果相邻子词间没有中止符，则将两子词直接拼接，否则两子词之间添加分隔符。

## WordPiece

Google的Bert模型在分词的时候使用的是WordPiece算法。与BPE算法类似，WordPiece算法也是每次从词表中选出两个子词合并成新的子词。与BPE的最大区别在于，如何选择两个子词进行合并：BPE选择频数最高的相邻子词合并，而WordPiece选择能够提升语言模型概率最大的相邻子词加入词表。



## Unigram Language Model (ULM)

与WordPiece一样，Unigram Language Model(ULM)同样使用语言模型来挑选子词。不同之处在于，BPE和WordPiece算法的词表大小都是从小到大变化，属于增量法。而Unigram Language Model则是减量法,即先初始化一个大词表，根据评估准则不断丢弃词表，直到满足限定条件。ULM算法考虑了句子的不同分词可能，因而能够输出带概率的多个子词分段。



## SentencePiece

如何使用上述子词算法？一种简便的方法是使用SentencePiece，它是谷歌推出的子词开源工具包，其中集成了BPE、ULM子词算法。除此之外，SentencePiece还能支持字符和词级别的分词。更进一步，为了能够处理多语言问题，sentencePiece将句子视为Unicode编码序列，从而子词算法不用依赖于语言的表示。