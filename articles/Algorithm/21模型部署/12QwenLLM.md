# vLLM部署

参数

```
--dtypefloat16 bfloat16
--max-model-len		模型支持的最大token数。一般在模型的config.json配置文件中有默认最大长度的配置。
--tensor-parallel-size  张量并行的数量
--pipeline-parallel-size 流水线并行的数量

--max-num-batched-tokens 每次迭代的batched的最大token数 None
--max-num-seqs 每次迭代的最大序列数 256
--max-paddings batch的最大填充数 256
```



# S-LoRA部署

https://github.com/S-LoRA/S-LoRA

