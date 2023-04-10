---
description: Can we finetune a LLM to get funny ?
tags: python LLM GPT finetune huggingface
img: i_have_achieved_comedy.jpg
comments: true
---


## Introduction

A lot of things have moved in the NLP space recently, ChatGPT ignited everything, but the best elements come from the capability to finetune some models at home.

Following [this post from HuggingFace], I wanted to test a finetuning idea : make a LLM understands and generate humor.

Currently, you get different behaviors with ChatGPT : 

![Screen from ChatGPT success](good_joke_chatgpt.png)[]
![Screen from ChatGPT failing](bad_joke_chatgpt.png)[]


But if you take an off the shelf model like GPT-Neo 2.7B, you don't get a joke easily : 

![Failed joke neo](joke_fail_neo_1.png)[]

It can even get weird : 


![weird joke neo](joke_fail_neo_2.png)[]


So the goal will be to train a LLM in the chatGPT way in order to generate jokes closer to a human one.


## 1 - RL trained LLM




## 2 - Train the reward function

#### 2-a) Collect the dataset

There is a reddit joke dataset with feedback collected.

One can easily adapt it as a HF daatset with the following code snippet


```python
# First retrieve the dataset from ......

import os

def rewrite_f_without_error(file_path, delimiter="\t", n_cols=2):
    dir_name = os.path.dirname(file_path)
    base_name = os.path.basename(file_path)
    fname = base_name.split(".")[0]
    file_ext = base_name.split(".")[1]
    with open(os.path.join(dir_name, fname + "_fixed." + file_ext), "w") as g:
        with open(file_path) as f:
            for l in f.readlines():
                n_split = len(l.split(delimiter))
                if n_split == n_cols:
                    g.write(l)

p1 = "/home/amor/Documents/code_dw/generative_joke/train.tsv"
p2 = "/home/amor/Documents/code_dw/generative_joke/test.tsv"
                    
rewrite_f_without_error(p1)
rewrite_f_without_error(p2)




file_dict = {
    "train": "/home/amor/Documents/code_dw/generative_joke/train_fixed.tsv",
    "test": "/home/amor/Documents/code_dw/generative_joke/test_fixed.tsv",
}

features = datasets.Features({'text': datasets.Value('string'), 'label': datasets.Value('float')})
dataset = datasets.load_dataset('csv', data_files=file_dict, delimiter='\t', 
                       column_names=['label', 'text'], 
                       features=features)
```

#### 2-b) Training of the model

As in the HF article, we want to use the new quantization technics in order to run everything on a single consumer grade GPU.

Everything can be found in the [PEFT]() package


- Tokenization step


```python

model_name = 'EleutherAI/gpt-neo-125M'

tokenizer = AutoTokenizer.from_pretrained(model_name)
if tokenizer.pad_token_id is None:
    tokenizer.pad_token_id = tokenizer.eos_token_id
#def tokenize_function(examples):
#    return tokenizer(examples["text"], padding="longest", truncation=True)
     

train_ds = build_dataset(tokenizer, dataset['train'])

test_ds = build_dataset(tokenizer, dataset['test'])

```

- Model preparation

```python
from peft import LoraConfig, PeftConfig, PeftModel, get_peft_model, prepare_model_for_int8_training




model = AutoModelForSequenceClassification.from_pretrained(model_name, 
                                                           num_labels=1,
                                                           load_in_8bit=True,
                                                           device_map='auto',
                                                           problem_type="regression",
                                                           ignore_mismatched_sizes=True)
model = prepare_model_for_int8_training(model)

config = LoraConfig(
    r=16, lora_alpha=32, target_modules=None, lora_dropout=0.05, bias="none", task_type="CAUSAL_LM"
)

model = get_peft_model(model, config)
model.gradient_checkpointing_enable()
model.config.use_cache = False  # silence the warnings. Please re-enable for inference!
model.config.pad_token_id = model.config.eos_token_id

model
```

- Model  training

```python
batch_size = 8

args = TrainingArguments(
    evaluation_strategy = "epoch",
    save_strategy = "epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=batch_size,
    per_device_eval_batch_size=batch_size,
    num_train_epochs=10,
    report_to="none",
    weight_decay=0.01,
    output_dir='./funny_finetune',
    )


trainer = Trainer(
    model,
    args,
    train_dataset=train_ds,
    eval_dataset=test_ds,
    tokenizer=tokenizer,
)

with torch.cuda.amp.autocast():
    trainer.train()
```

It works reasonably well and doesn't take too much VRAM : 

![Model perf](model_training_screen.png){}

![Model memory](gpu_consumption.png){}



## 3) Testing the model




