https://zhuanlan.zhihu.com/p/678613724



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

