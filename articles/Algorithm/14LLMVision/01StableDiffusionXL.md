时间：2023.07.27

机构：Stability AI

与Stable DiffusionV1-v2相比，Stable Diffusion XL主要做了如下的优化：

1. 对Stable Diffusion原先的U-Net，VAE，CLIP Text Encoder三大件都做了改进。
2. 增加一个单独的基于Latent的Refiner模型，来提升图像的精细化程度。
3. 设计了很多训练Tricks，包括图像尺寸条件化策略，图像裁剪参数条件化以及多尺度训练等
4. **先发布Stable Diffusion XL 0.9测试版本，基于用户使用体验和生成图片的情况，针对性增加数据集和使用RLHF技术优化迭代推出Stable Diffusion XL 1.0正式版**。



# 核心知识

## 整体架构

Stable Diffusion XL是一个**二阶段的级联扩散模型**，包括Base模型和Refiner模型。其中Base模型的主要工作和Stable Diffusion一致，具备文生图，图生图，图像inpainting等能力。在Base模型之后，级联了Refiner模型，对Base模型生成的图像Latent特征进行精细化，**其本质上是在做图生图的工作**。

**Base模型由U-Net，VAE，CLIP Text Encoder（两个）三个模块组成**，在FP16精度下Base模型大小6.94G（FP32：13.88G），其中U-Net大小5.14G，VAE模型大小167M以及两个CLIP Text Encoder一大一小分别是1.39G和246M。

**Refiner模型同样由U-Net，VAE，CLIP Text Encoder（一个）三个模块组成**，在FP16精度下Refiner模型大小6.08G，其中U-Net大小4.52G，VAE模型大小167M（与Base模型共用）以及CLIP Text Encoder模型大小1.39G（与Base模型共用）。

![img](images/v2-4f2789da0edce3064fcb34eeac0104de_720w.webp)

## U-Net

U-Net的Encoder和Decoder结构也从原来的4stage改成3stage（[1,1,1,1] -> [0,2,10]），说明SDXL只使用两次下采样和上采样，而之前的SD系列模型都是三次下采样和上采样。并且比起Stable DiffusionV1/2，Stable Diffusion XL在第一个stage中不再使用Spatial Transformer Blocks，而在第二和第三个stage中大量增加了Spatial Transformer Blocks（分别是2和10）

## VAE

**Stable Diffusion XL使用了和之前Stable Diffusion系列一样的VAE结构**，但在训练中选择了**更大的Batch-Size（256 vs 9）**，并且对模型进行指数滑动平均操作（**EMA**，exponential moving average），EMA对模型的参数做平均，从而提高性能并增加模型鲁棒性。

在损失函数方面，使用了久经考验的**生成领域“交叉熵”—感知损失（perceptual loss）**以及回归损失来约束VAE的训练过程。

与此同时，VAE的**缩放系数**也产生了变化。VAE在将Latent特征送入U-Net之前，需要对Latent特征进行缩放让其标准差尽量为1，之前的Stable Diffusion系列采用的**缩放系数为0.18215，**由于Stable Diffusion XL的VAE进行了全面的重训练，所以**缩放系数重新设置为0.13025**。

注意：由于缩放系数的改变，Stable Diffusion XL VAE模型与之前的Stable Diffusion系列并不兼容。



## CLIP Text Encoder

**CLIP模型主要包含Text Encoder和Image Encoder两个模块**，在Stable Diffusion XL中，和之前的Stable Diffusion系列一样，**只使用Text Encoder模块从文本信息中提取Text Embeddings**。

**不过Stable Diffusion XL与之前的系列相比，使用了两个CLIP Text Encoder，分别是OpenCLIP ViT-bigG（1.39G）和OpenAI CLIP ViT-L（246M），从而大大增强了Stable Diffusion XL对文本的提取和理解能力。**

**与传统深度学习中的模型融合类似**，Stable Diffusion XL分别提取两个Text Encoder的倒数第二层特征，并进行concat操作作为文本条件（Text Conditioning）。其中OpenCLIP ViT-bigG的特征维度为77x1280，而CLIP ViT-L的特征维度是77x768，所以输入总的特征维度是77x2048（77是最大的token数），再通过Cross Attention模块将文本信息传入Stable Diffusion XL的训练过程与推理过程中。

和Stable Diffusion一致的是，Stable Diffusion XL输入的最大Token数依旧是77，当输入text的Token数量超过77后，将通过Clip操作拉回77；如果Token数不足77则会padding操作得到77x2048。

## Refiner

由于已经有Base模型生成了图像的Latent特征，所以**Refiner模型的主要工作是在Latent特征进行小噪声去除和细节质量提升**。

和Base兼容同一个VAE模型，不过Refiner模型的Text Encoder只使用了OpenCLIP ViT-bigG。

## 训练技巧&细节

