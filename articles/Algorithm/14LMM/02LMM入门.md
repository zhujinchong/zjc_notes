# BLIP

**BLIP**(Bootstrapping Language-Image Pretraining)是**salesforce**在2022年提出的多模态框架，是理解和生成的统一，引入了跨模态的编码器和解码器，实现了跨模态信息流动，在多项视觉和语言任务取得SOTA。

## 模型结构

**BLIP引入了编码器-解码器的多模态混合结构MED**（ Multimodal mixture of Encoder-Decoder）**，能够有效地进行多任务预学习和迁移学习。**MED包括两个单模态编码器（lmage Encoder，Text Encoder），一个以图像为基础的编码器（image-grounded text encoder）和一个以图像为基础的解码器（image-grounded text decoder）。

