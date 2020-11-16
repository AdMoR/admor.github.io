---
description: Essential ressources to use for fun and profit
tags: NLP transformers text generation
img: couverture/transformer.jpg
---

# Why tranformers ?

Published in 2018, the transformer architecture was a breakthrought for NLP tasks, providing massive uplift compared to previous models.

However even if the paper [Attention is all you need](https://arxiv.org/pdf/1706.03762.pdf) is well written, it takes time to understand all the subtilities of the model.

This post is my personal cheatsheet for this kind of models.


### Understanding the architecture

Rather than repetiting something that has already been done better, here are the links for understanding the model that I found the most helpful.

- [The illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) : great article with insightful visualization. Other NLP articles are also available on the same blog.
- [The annotated Transformer](http://nlp.seas.harvard.edu/2018/04/03/attention.html) : Detailled implementation of the model, step by step. Really insightful to understand the details of the architecture


### Getting started to experiment

I would recommend using HuggingFace great repository on Github to play with this kind of architecture as they have all the state of the art models

[HuggingFace's Transformers](https://github.com/huggingface/transformers)

They support both Pytorch and Tensorflow, so everyone gets what he needs.


# Model size and solving one's problem

The problem of modern language model is their size and the scale needed to train them.
The most recent models may need a complete cluster equipped with powerful graphic cards to be trained.
These models may not always perform the best for the task that you want to solve.


Here are some insights on what can be done to save on your AWS bill : 

- Bag of words models remain relatively comptetitve according to this [study](https://arxiv.org/pdf/1806.06259.pdf). When little data is available, they are a more than satisfactory solution and easy to implement.

- [Distillation](https://arxiv.org/abs/1910.01108) can help to reduce the model size and improve the inference speed, while keeping most of the performance. You can then train your distilled model if you need to.


