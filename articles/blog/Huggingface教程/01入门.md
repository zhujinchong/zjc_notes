# 入门

huggingface官网教程

```
https://huggingface.co/learn
```

安装

```
pip install transformers
```

## pipline函数

pipline把所有东西都封装起来，一个函数即可用，但实际不会这么用。

```
from transformers import pipeline


# 情感分析
classifier = pipeline("sentiment-analysis")
classifier("I've been waiting for a HuggingFace course my whole life.")
# 文本生成
# model：指定模型
generator = pipeline("text-generation", model="distilgpt2")
generator("In this course, we will teach you how to", max_length=30)
# 命名实体识别
ner = pipeline("ner", grouped_entities=True)
ner("My name is Sylvain and I work at Hugging Face in Brooklyn.")
```

## Model

加载模型，模型参数随机初始化，即，未训练的模型：

```
from transformers import BertConfig, BertModel

config = BertConfig()
model = BertModel(config)
print(config)	# config定义了模型结构
```

从checkpoint加载模型：

```
model = BertModel.from_pretrained("bert-base-cased")
```

模型保存：

```
model.save_pretrained("directory_on_my_computer")
# 保存后会有两个文件：
# config.json 配置+模型结构
# pytorch_model.bin 权重字典
```

## Tokenizer

input数据需要变成batch形式，才可以输入模型。

```
import torch
from transformers import AutoTokenizer, AutoModelForSequenceClassification

checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(checkpoint)
model = AutoModelForSequenceClassification.from_pretrained(checkpoint)

sequence = "I've been waiting for a HuggingFace course my whole life."
tokens = tokenizer.tokenize(sequence)
ids = tokenizer.convert_tokens_to_ids(tokens)
input_ids = torch.tensor([ids])
print(input_ids)
# [[ 1045,  1005,  2310,  2042,  3403,  2005,  1037, 17662, 12172,  2607, 2026,  2878,  2166,  1012]]

output = model(input_ids)
print("Logits:", output.logits)
# [[-2.7276,  2.8789]]
```

batch填充：tokenizer中都有pad_token_id

```
batched_ids = [
    [200, 200, 200],
    [200, 200, tokenizer.pad_token_id],
]
```

自动填充：进一步简化代码

```
# 按最长序列填充 （建议）
model_inputs = tokenizer(sequences, padding="longest")

# 模型支持多长，填充到最长（不建议）
model_inputs = tokenizer(sequences, padding="max_length")

# 填充到自定义的长度（不建议）
model_inputs = tokenizer(sequences, padding="max_length", max_length=8)
```

句子截断

```
# 超过最大长度，就会截断
model_inputs = tokenizer(sequences, truncation=True)
```

自动返回张量：简化代码，不用再写转换张量代码

```
# Returns PyTorch tensors
model_inputs = tokenizer(sequences, padding=True, return_tensors="pt")

# Returns TensorFlow tensors
model_inputs = tokenizer(sequences, padding=True, return_tensors="tf")
```

自动填充特殊token：比如BERT会在句子开头和模型添加特殊字符，此时Tokenizer也做了

```
model_inputs = tokenizer(sequence)
print(tokenizer.decode(model_inputs["input_ids"])) # 发现tokenizer自动做了特殊字符填充

tokens = tokenizer.tokenize(sequence)
ids = tokenizer.convert_tokens_to_ids(tokens)
print(tokenizer.decode(ids))  # 自己调用 convert_tokens_to_ids并不会填充

# "[CLS] i've been waiting for a huggingface course my whole life. [SEP]"
# "i've been waiting for a huggingface course my whole life."
```

总结：

```
# 填充、截断、返回张量
tokens = tokenizer(sequences, padding=True, truncation=True, return_tensors="pt")
output = model(**tokens)
```

## 准备数据集

加载数据集：此命令下载并缓存数据集，默认位于 ~/.cache/huggingface/datasets 中。您可以通过设置 HF_HOME 环境变量来自定义缓存文件夹。

```
from datasets import load_dataset

raw_datasets = load_dataset("glue", "mrpc")
raw_datasets
```

批量转换数据集

```
def tokenize_function(example):
    return tokenizer(example["sentence1"], example["sentence2"], truncation=True)


raw_datasets = load_dataset("glue", "mrpc")
tokenized_datasets = raw_datasets.map(tokenize_function, batched=True)
```

动态填充**dynamic padding**

```
from transformers import DataCollatorWithPadding

data_collator = DataCollatorWithPadding(tokenizer=tokenizer)
samples = tokenized_datasets["train"][:8]
batch = data_collator(samples)
```



## 微调

Trainer类就是来帮你微调的，你需要：

1、准备数据

2、定义参数TrainingArguments

3、开启微调trainer.train()

```
from datasets import load_dataset
from transformers import AutoTokenizer, DataCollatorWithPadding, TrainingArguments, AutoModelForSequenceClassification, Trainer
import evaluate


checkpoint = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(checkpoint)
# 注意1：这里修改了BERT模型的输出，定义了分类输出，会出现警告日志
model = AutoModelForSequenceClassification.from_pretrained(checkpoint, num_labels=2)

raw_datasets = load_dataset("glue", "mrpc")
tokenized_datasets = raw_datasets.map(lambda example: tokenizer(example["sentence1"], example["sentence2"], truncation=True), batched=True)
data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

# 注意2：训练和评估的超参数（这里保留默认）
training_args = TrainingArguments(checkpoint, evaluation_strategy="epoch")


# 注意3：这里定义了一个评估方法，参数就是模型的输出
def compute_metrics(eval_preds):
    metric = evaluate.load("glue", "mrpc")
    logits, labels = eval_preds
    predictions = np.argmax(logits, axis=-1)
    return metric.compute(predictions=predictions, references=labels)


trainer = Trainer(
    model,
    training_args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["validation"],
    data_collator=data_collator,
    tokenizer=tokenizer,
    compute_metrics=compute_metrics,
)
trainer.train()
```



## 训练

```
from datasets import load_dataset
from transformers import AutoTokenizer, DataCollatorWithPadding
from torch.utils.data import DataLoader
from transformers import AdamW, get_scheduler
from tqdm.auto import tqdm
import evaluate


device = torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")
checkpoint = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(checkpoint)
model = AutoModel.from_pretrained(checkpoint, num_labels=2)
model.to(device)

raw_datasets = load_dataset("glue", "mrpc")
tokenized_datasets = raw_datasets.map(lambda example: tokenizer(example["sentence1"], example["sentence2"], truncation=True), batched=True)
# 删除不用的列
tokenized_datasets = tokenized_datasets.remove_columns(["sentence1", "sentence2", "idx"])
# 重命名标签
tokenized_datasets = tokenized_datasets.rename_column("label", "labels")
# 转为张量
tokenized_datasets.set_format("torch")
# 动态填充
data_collator = DataCollatorWithPadding(tokenizer=tokenizer)
# 加载器
train_dataloader = DataLoader(
    tokenized_datasets["train"], shuffle=True, batch_size=8, collate_fn=data_collator
)
eval_dataloader = DataLoader(
    tokenized_datasets["validation"], batch_size=8, collate_fn=data_collator
)

# 优化器
optimizer = AdamW(model.parameters(), lr=5e-5)
num_epochs = 3
# 学习率线性衰减至0
num_training_steps = num_epochs * len(train_dataloader)
lr_scheduler = get_scheduler(
    "linear",
    optimizer=optimizer,
    num_warmup_steps=0,
    num_training_steps=num_training_steps,
)

progress_bar = tqdm(range(num_training_steps))
for epoch in range(num_epochs):
    model.train()
    for batch in train_dataloader:
        batch = {k: v.to(device) for k, v in batch.items()}
        outputs = model(**batch)
        loss = outputs.loss
        loss.backward()

        optimizer.step()
        lr_scheduler.step()
        optimizer.zero_grad()
        progress_bar.update(1)

    model.eval()
    metric = evaluate.load("glue", "mrpc")
    for batch in eval_dataloader:
        batch = {k: v.to(device) for k, v in batch.items()}
        with torch.no_grad():
            outputs = model(**batch)
        logits = outputs.logits
        predictions = torch.argmax(logits, dim=-1)
        metric.add_batch(predictions=predictions, references=batch["labels"])
    metric.compute()
```



# 数据集详解

## 数据处理

加载本地数据集

```
from datasets import load_dataset

!wget https://github.com/crux82/squad-it/raw/master/SQuAD_it-train.json.gz
!wget https://github.com/crux82/squad-it/raw/master/SQuAD_it-test.json.gz
raw_data = load_dataset("json", data_files="SQuAD_it-train.json", field="data")
```

打乱和选择

```
drug_sample = drug_dataset["train"].shuffle(seed=42).select(range(1000))
drug_sample[:3]
```

过滤None数据

```
drug_dataset = drug_dataset.filter(lambda x: x["condition"] is not None)
```

创建新列

```
drug_dataset = drug_dataset.map(lambda x: {"review_length": len(x["review"].split())})
```

通过batch操作，加速

```
tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")


def tokenize_and_split(examples):
    return tokenizer(examples["review"], truncation=True, max_length=128, return_overflowing_tokens=True)


tokenized_dataset = drug_dataset.map(tokenize_and_split, batched=True)
```

上面会报错，因为return_overflowing_tokens会导致多出几列，所以还要删除旧列。

删除所有旧列

```
tokenized_dataset = drug_dataset.map(
    tokenize_and_split, batched=True, remove_columns=drug_dataset["train"].column_names
)
```

## 数据流

几百G数据怎么处理，流失处理：

```
pubmed_dataset_streamed = load_dataset(
    "json", data_files=data_files, split="train", streaming=True
)
next(iter(pubmed_dataset_streamed))
```

打乱数据

```
shuffled_dataset = pubmed_dataset_streamed.shuffle(buffer_size=10_000, seed=42)
next(iter(shuffled_dataset))
```





# PEFT

## 微调代码

```
from peft import LoraConfig, TaskType, get_peft_model
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer


checkpoint = "bigscience/mt0-large"
model = AutoModelForSeq2SeqLM.from_pretrained(checkpoint)
tokenizer = AutoTokenizer.from_pretrained(checkpoint)

# peft
peft_config = LoraConfig(task_type=TaskType.SEQ_2_SEQ_LM,
                         inference_mode=False,
                         r=8,
                         lora_alpha=32,
                         lora_dropout=0.05)
model = get_peft_model(model, peft_config)
model.print_trainable_parameters()

train_dataset=
eval_dataset=
data_collator=

trainer = LoRATrainer(
    model=model,
    args=hf_train_args,
    train_dataset=tfrom peft import LoraConfig, TaskType, get_peft_model
trainer.train()
# 保存得到 adapter_config.json adapter_model.bin
trainer.model.save_pretrained(output_dir)
```

## 模型加载

加载peft配置，加载base模型，加载peft+base模型

```
from transformers import AutoModelForSeq2SeqLM
from peft import PeftModel, PeftConfig


peft_checkpoint = "smangrul/twitter_complaints_bigscience_T0_3B_LORA_SEQ_2_SEQ_LM"
peft_config = PeftConfig.from_pretrained(peft_checkpoint)
base_model = AutoModelForSeq2SeqLM.from_pretrained(peft_config.base_model_name_or_path).cuda()
model = PeftModel.from_pretrained(base_model, peft_checkpoint)
tokenizer = AutoTokenizer.from_pretrained(peft_config.base_model_name_or_path)
model.eval()

while True:
    try:
        query = input("\nInput: ")
    except Exception:
        raise
    response, history = model.chat(tokenizer=tokenizer, query=query)
    print("Output: " + response)
```

也支持一个方法完成加载

```
from peft import AutoPeftModelForCausalLM
peft_model = AutoPeftModelForCausalLM.from_pretrained("ybelkada/opt-350m-lora")
```



## 模型导出



## QLora

