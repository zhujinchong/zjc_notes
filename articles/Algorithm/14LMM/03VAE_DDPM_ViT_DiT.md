主流生成模型：

![img](images/v2-29563658cff776aee4fb49d84eec2741_720w.webp)

# VAE

https://blog.csdn.net/v_JULY_v/article/details/130361959?spm=1001.2014.3001.5502



AE自编码器存在的问题：绝大多数的随机的潜空间z是没有任何意义的噪声。原因在于没有显性的对z的分布进行建模。

VAE(自变分编码器，Variational Autoencoders)则是在AE的基础上，**显性的对![z](images/eq-1708348696726.png)的分布![p(images/eq.png)](https://latex.csdn.net/eq?p%28z%29)进行建模**(比如符合某种常见的概率分布)**，使得自编码器成为一个合格的生成模型**。

为了抑制自编码过程中的过拟合，VAE编码器的输出是一个正态分布，而不是一个具体的编码。VAE的损失函数除了要最小化重建图像与原图像之间的均方误差外，还要最大化每个分布和标准正态分布之间的相似度。

最终表现形式就是在目标函数Loss中加正则化：

![img](images/v2-82ba828b4f6fb7e0a9aed4495ff407ac_720w-1708603386528.webp)



代码实现：https://www.cnblogs.com/picassooo/p/12601785.html



# DDPM

https://blog.csdn.net/v_JULY_v/article/details/130361959?spm=1001.2014.3001.5502

DDPM（Denoising Diffusion Probabilistic Models）主要有两个贡献

 1、**从预测转换图像改进为预测噪声** (即如DiT论文所说，reformulating diffusion models to predict noise instead of pixel，可惜强调这点的文章太少了，可它是DDPM的关键，更是DDPM的本质）DDPM采用了一个U-Net 结构的Autoencoder对t时刻的高斯噪声z进行预测。

2、**DDPM只预测正态分布的均值**，虽然正态分布由均值和方差决定，但作者在这里发现，其实模型不需要学方差，只需要学习均值就行。逆向过程中高斯分布的方差项直接使用一个常数，模型的效果就已经很好。所以就再一次降低了模型的优化难度。





# DETR



# ViT

