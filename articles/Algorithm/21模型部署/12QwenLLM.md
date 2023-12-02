

# 如何加速

1、batch推理

2、flash-attention（fp16或bf16）

2、kv cache缓存

3、模型量化：用AutoGPTQ技术，得到BF16, Int8和Int4模型

4、多卡推理：vLLM和FastChat



注意：KV Cache和FlashAttention不能同时用，同时开启时，FlashAttention关闭。

推荐：KV

