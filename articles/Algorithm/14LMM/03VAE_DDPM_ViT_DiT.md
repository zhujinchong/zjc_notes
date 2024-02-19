# VAE

AE自编码器存在的问题：绝大多数的随机的潜空间z是没有任何意义的噪声。原因在于没有显性的对z的分布进行建模。

VAE(自变分编码器，Variational Autoencoders)则是在AE的基础上，**显性的对![z](images/eq-1708348696726.png)的分布![p(images/eq.png)](https://latex.csdn.net/eq?p%28z%29)进行建模**(比如符合某种常见的概率分布)**，使得自编码器成为一个合格的生成模型**。



https://blog.csdn.net/v_JULY_v/article/details/130361959

https://blog.csdn.net/v_JULY_v/article/details/136143475