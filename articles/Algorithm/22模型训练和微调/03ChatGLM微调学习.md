# chatGLM-6b-QLora

## 微调环境

容器，用QLora

```
# 拉镜像
docker pull huggingface/transformers-pytorch-gpu:4.29.1
# 删除镜像
docker rmi huggingface/transformers-pytorch-gpu:4.29.1

# 启动容器; 端口用于查看训练曲线
docker run -tid -v /data/zjc/docker:/home -p 58323:58323 --name zjchf huggingface/transformers-pytorch-gpu:4.29.1
# 删除容器 停止容器 启动容器
docker rm zjchf
docker stop zjchf
docker start zjchf

# 进入容器
docker exec -it zjchf /bin/bash

# 安装包时网络连接失败，重启docker解决：systemctl restart docker
python3 -m pip install --upgrade pip
pip install -q -U bitsandbytes
pip install -q -U git+https://github.com/huggingface/peft.git
pip install transformers==4.30.2
pip install accelerate==0.20.3

```



微调

```
# 容器中训练
python3 train_qlora.py \
--train_args_json /home/qlora_chatglm6b/chatGLM_6B_QLoRA.json \
--model_name_or_path /home/chatglm2-6b-f16 \
--train_data_path data/train.jsonl \
--lora_rank 4 \
--lora_dropout 0.05 \
--compute_dtype fp16

python3 train_qlora.py \
--train_args_json /home/chatGLM-6B-QLoRA-main/chatGLM_6B_QLoRA.json \
--model_name_or_path /home/chatglm2-6b-f16 \
--train_data_path data/self_cognition_sai.json \
--lora_rank 8 \
--lora_dropout 0.05 \
--compute_dtype fp16

# 主机中启动tensorboard
tensorboard --logdir runs --port '222' --host 127.0.0.1

# 容器中测试训练效果
python3 cli_demo.py
```

模型导出

```
python3 merge_lora_and_quantize.py \
--lora_path saved_files/chatGLM_6B_QLoRA_t32 \
--output_path export_model/merged_qlora_model_4bit \
--remote_scripts_dir remote_scripts/chatglm2-6b \
--qbits 4
```



## 设置显卡

一定要在文件**最开头**设置 

```
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "0"
os.environ["CUDA_VISIBLE_DEVICES"] = "1,2"
```



# ChatGLM-Efficient-Tuning

训练

```bash
CUDA_VISIBLE_DEVICES=0 python src/train_bash.py \
    --stage sft \
    --do_train \
    --dataset self_cognition_sai \
    --finetuning_type lora \
    --output_dir cognition \
    --overwrite_cache \
    --per_device_train_batch_size 4 \
    --gradient_accumulation_steps 2 \
    --lr_scheduler_type cosine \
    --logging_steps 10 \
    --save_steps 1000 \
    --warmup_steps 0 \
    --learning_rate 1e-3 \
    --num_train_epochs 10.0 \
    --fp16 \
    --model_name_or_path /home/chatglm2-6b-f16
```

测试

```bash
CUDA_VISIBLE_DEVICES=0 python3 src/cli_demo.py \
    --checkpoint_dir cognition \
    --model_name_or_path /home/chatglm2-6b-f16
```

模型导出

```bash
CUDA_VISIBLE_DEVICES=0 python3 src/export_model.py \
    --checkpoint_dir cognition \
    --output_dir export_model/chatglm2-lora \
    --model_name_or_path /home/chatglm2-6b-f16
```