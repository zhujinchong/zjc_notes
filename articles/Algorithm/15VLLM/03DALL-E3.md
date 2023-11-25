时间：2023.10

机构：OpenAI

技术报告：Improving Image Generation with Better Captions



 一句话省流版，数据方面，训练时使用95%模型（CoCa）合成详细描述caption + 5%原本人类 caption，测试时使用GPT-4v 扩写人类caption；模型方面使用T5xxl + vae encoder + diffusion latent + 自家decoder 取得最好效果。

# 合成caption

