https://zhuanlan.zhihu.com/p/678613724

# Dreambooth

2022-04



背景：

文生图模型可以根据prompt生成高质量图片，但是模型不能保留ID，在不同情境中生成新颖的图片。（即主角不变，只改变背景）。

DreamBooth是一种个性化文生图模型：给定某个物体的几张图片作为输入，通过微调预训练的文生图模型（如Imagen），将一个独特的标识符和该物体进行绑定，这样就可以通过含有该标识符的prompt在不同场景下生成包含该物体的新颖图片。

![image-20240220223300204](images/image-20240220223300204.png)

方法：

作者希望将输入图片中的物体与一个特殊标识符绑定在一起，即用这个特殊标记符来表示输入图片中的物体。因此作者为微调模型设计了一种prompt格式：`a [identifier] [class noun]`，即将所有输入图片的promt都设置成这种形式，其中`identifier`是一个与输入图片中物体相关联的特殊标记符，`class noun`是对物体的类别描述。这里之所以在prompt中加入类别，是因为作者想利用预训练模型中关于该类别物品的先验知识，并将先验知识与特殊标记符相关信息进行融合，这样就可以在不同场景下生成不同姿势的目标物体。

例如：

- 3-10张图片, 最好是不同角度，且背景有变化的图片
- 类名class name+独特标识符unique identifier，例如：instance prompt: a photo of **Halley** Dog; class prompt: a photo of **Dog**。这里Halley是独特标识符，Dog是类名。
- 微调即可。



# Textual Inversion

时间：2022-08



> **embedding是textual inversion的结果，因此textual inversion也可以称为embedding。该方法只需要3-5张图像，通过定义新的关键词，就能生成和训练图像相似的风格。**



背景：Stable Diffusion它们的Embedding空间的表现力足以捕捉基本的语义信息，但是这两者都无法准确捕捉概念的外观，将它们用于特定主题的图像生成会导致相当大的视觉差异。如果为每个新概念使用特定数据集重新训练模型，那成本将非常高，并且对少数示例图片进行finetune通常会导致灾难性的遗忘。

本文提出Textual Inversion方法，只需使用用户提供的3~5张概念图片，通过学习文图生成模型Text Embedding空间中的伪词（pseudo-word）来表示这些概念。然后把这些伪词组合成自然语言的句子，指导个性化生成。

![image-20240220224418797](images/image-20240220224418797.png)

原理：

- 输入字符串中的每个词（**word**）或子词（**sub-word**）都被转换为一个标记（**Token**），它是预定义词典中的索引（参见**BPE**算法）；
- 然后将每个**Token**对应到一个唯一的嵌入向量（**embedding**），这些嵌入向量通常作为文本编码器的一部分进行学习；
- 选择这个嵌入向量空间作为反演（**Inversion**）的目标，指定一个占位符字符串 **S\*** 来表示希望学习的新概念；
- 用新学习的嵌入向量 **v\*** 替换与新概念关联的嵌入向量，即将概念注入（**inject**）到词汇表中；
- 跟处理其它单词一样，用该概念字符组成新句子，例如： A photo of **S\***, A painting in the style of **S\*.**



因此，Textual Inversion只需要训练文本嵌入层即可，而Dreambooth理念需要微调文生图模型。



# Hypernetworks

hypernetworks是一种fine tune的技术，最开始由novel AI开发。hypernetworks是一个附加到stable diffusion model上的小型网络，用于修改扩散模型的风格。

既然Hypernetworks会附加到diffusion model上，那么会附加到哪一部分呢？答案仍然是UNet的cross-attention模块，Lora模型修改的也是这一部分，不过方法略有不同。

hypernetworks是一个很常见的神经网络结构，具体而言，是一个带有dropout和激活函数的全连接层。通过插入两个网络转换key和query向量。

![image-20240220225241164](images/image-20240220225241164.png)

和其他方法的对比：

LoRA: Lora模型和Hypernetworks比较相似，都是通过作用于UNet的cross-attention模块，改变生成图像的风格。区别在于LoRA改变的是cross-attention的权重，而Hypernetworks插入了其他的模块。LoRA的结果通常比Hypernetworks更好，而且模型结构都很小，基本都低于200Mb。值得注意的是，LoRA指的是一种数据存储的方式，没有定义训练过程，因此可以和Dreambooth和其他的训练方式结合起来使用。

embeddings: 是textual inversion方法的结果，和Hypernetworks方法一样，不会改变模型，会定义一些新的关键字实现某些样式。embedding和Hypernetworks作用的地方不同。embedding在text encoder中创造新的embedding达到左右模型生成图片的效果。根据一些参考资料，embedding的效果比Hypernetworks要稍好一些。

# ControlNet

时间：2023-02



ControlNet可以从提供的参考图中获取布局或者姿势等信息, 并引导diffusion model生成和参考图类似的图片。调整prompt确实是一件费时费力的事情，而有了ControlNet则可以精确的控制生成图片的细节(当然, 也依赖于你已经有一张参考图了)。



# 一、IP-Adapter

# 二、PhotoMaker

# 三、InstantID

https://mp.weixin.qq.com/s/7d8En7idif4UFSGo_6MFsg

主题驱动的文本到图像生成，通常需要在多张包含该主题（如人物、风格）的数据集上进行训练，这类方法中的代表工作包括 DreamBooth、Textual Inversion、LoRAs 等，但这类方案因为需要更新整个网络或较长时间的定制化训练，往往无法很有效地兼容社区已有的模型，并无法在真实场景中快速且低成本应用。而目前基于单张图片特征进行嵌入的方法（FaceStudio、PhotoMaker、IP-Adapter），要么需要对文生图模型的全参数训练或 PEFT 微调，影响原本模型的泛化性能，缺乏与社区预训练模型的兼容性，要么无法保持高保真度。

InstantID 是一个高效的、轻量级、可插拔的适配器，赋予预训练的文本到图像扩散模型以 ID 保存的能力。作者通过:

（1）将弱对齐的 CLIP 特征替换为强语义的人脸特征；

（2）人脸图像的特征在 Cross-Attention 中作为 Image Prompt 嵌入；

（3）提出 IdentityNet 来对人脸施加强语义和弱空间的条件控制，从而增强 ID 的保真度以及文本的控制力。



# 四、AnyDoor

https://mp.weixin.qq.com/s/vZAfcDlgCIi1B5gJNK8Vlg

