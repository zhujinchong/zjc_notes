时间：2022.04.13

机构：OpenAI

论文：Hierarchical Text-Conditional Image Generation with CLIP Latents



# 总览

它主要包括三个部分：CLIP，先验模块prior和img decoder。其中CLIP又包含text encoder和img encoder

![img](images/v2-2cd0e865cb388ba9a2093fc934aacaea_720w.webp)

# 训练过程

DALL·E 2是将其子模块分开训练的，最后将这些训练好的子模块拼接在一起，最后实现由文本生成图像的功能。

**1. 训练CLIP，使其能够编码文本和对应图像**

这一步是与CLIP模型的训练方式完全一样的，目的是能够得到训练好的text encoder和img encoder。这么一来，文本和图像都可以被编码到相应的特征空间。对应上图中的虚线以上部分。

**2. 训练prior，使文本编码可以转换为图像编码**

实际的训练过程为：将CLIP中训练好的text encoder拿出来，输入文本y，得到文本编码zt。同样的，将CLIP中训练好的img encoder拿出来，输入图像 x得到图像编码zi。我们希望prior能从zt获取相对应的zi。假设zt经过prior输出的特征为zi'，那么我们自然希望zi′与zi越接近越好，这样来更新我们的prior模块。最终训练好的prior，将与CLIP的text encoder串联起来，它们可以根据我们的输入文本y生成对应的图像编码特征zi了。

在DALL·E 2 模型中，作者团队尝试了两种先验模型：自回归式Autoregressive (AR) prior 和扩散模型Diffusion prior [1]。实验效果上发现两种模型的性能相似，而因为扩散模型效率较高，因此最终选择了扩散模型作为prior模块。

![img](images/v2-9735490632574f482139700d69156f78_720w.webp)

**3. 训练decoder生成最终的图像**

这个过程与自编码器类似，从中间特征层还原出输入图像，但又不完全一样。我们需要生成出的图像，只需要保持原始图像的显著特征就可以了，这样以便于多样化生成，例如下图右边的示例。

DALL-E 2使用的是改进的GLIDE模型 [2]。这个模型可以根据CLIP图像编码的zi，还原出具有相同与 zi 有相同语义，而又不是与 x 完全一致的图像。

# 推理

经过以上三个步骤的训练，已经可以完成DALL·E 2预训练模型的搭建了。我们这事丢掉CLIP中的img encoder，留下CLIP中的text encoder，以及新训练好的prior和decoder。这么一来流程自然很清晰了：由text encoder将文本进行编码，再由prior将文本编码转换为图像编码，最后由decoder进行解码生成图像。

![img](images/v2-47165c314858534d1be906db3c6ded36_720w.webp)

