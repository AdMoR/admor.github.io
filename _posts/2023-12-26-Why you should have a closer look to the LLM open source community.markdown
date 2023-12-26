---
description: The open source hides some true gems
tags: python LLM GenerativeAI
img: future_is_now.png
comments: true
---

You want to know what will come next in the LLM world. 
I may have an answer for you based on what happen with Dall-E and STable Diffusion.


## The Parallel with Dall-e

If you look back about 1.5 years ago and you wanted to use these latest txt-2-img models. You did not have so many choices : 
Dall-e, Midjourney or Stable diffusion.

After trying some of them, the visual results were striking :

![Dall-e vs stable diff]({{site.baseurl}}/assets/img/dalee_vs_stablediff.png)

With Dall-e, you could clearly see you had a better image quality.
As Dall-e was a commercial product, being actively developed. It would obviously win in the long term, right ?


However, in between, the open source developers started to play with Stable diffusion model and train lora on custom dataset.
These finetunings allowed to exceed the perceived state of the art image quality but also to represent some concept that the base model could not express.

![knuckles.png]({{site.baseurl}}/assets/ugandan_knuckles.png)

With the more recent releases of Stable-diffusion-XL, the gap between proprietary and open-source models remains limited.

![knuckles.png]({{site.baseurl}}/assets/midjou_vs_sdxl.png)

The main limitation today blocking a larger usage of Stable diffusion (SD) is the GPU requirement. 
When running on CPU, it runs 10 times slower and becomes less usable than any other cloud alternative.


## I thought we would talk about LLM 

Today, in the LLM landscape, we are not so far away from the stable diffusion use case from 1.5 years ago.

- 2 big proprietary player : ChatGPT and Claude
- Many smaller open source initiatives : Llama, Mistral and other

From a user perspective, we have the same statements : a cloud offer is easier to use, 
but when the proprietary models fail on your task, you may want to [finetune an open source model](https://medium.com/@dave-shap/a-pros-guide-to-finetuning-llms-c6eb570001d3)
or use a [Retrieval Augmented Generation](https://www.linkedin.com/posts/waleedkadous_fine-tuning-is-for-form-not-facts-anyscale-activity-7101638298120421377-66SA/).


## Where should I have a look then ?

#### r/localllama

How to better introduce this subreddit than with this quote from Karpathy ?

![knuckles.png]({{site.baseurl}}/assets/localllama.png)

This subreddit focuses on harnessing the open source LLM and making.
And things can get technical really quickly, even for machine learning practitioners.

![knuckles.png]({{site.baseurl}}/assets/is_this_satire.png)

I assure you this is not satire and an actual feedback from someone who tested many different models.

What to expect from this subreddit ?
- Practitioner advices on RAG and finetuning best practices
- Benchmarks to compare models
- Tests in production performance


#### Huggingface model repository

A lot of people are experimenting with models thanks to huggingface. 
But isolated experiments wouldn't bring as much as the model repository that Huggingface is today.

If we come back to the parallel of txt-2-img, finetuning on custom dataset have been used to learn how to generate a better base model.
You could find pretty advanced discussions on the model merging topic on reddit [1](https://www.reddit.com/r/StableDiffusion/comments/11sqsk8/merge_models_by_layers_and_similarity/), [2](https://www.reddit.com/r/StableDiffusion/comments/18ov9cg/comparison_opendalle_sdxl_model_merged_with_dpo/)

You can find a similar trend on HuggingFace for the LLM. Mistral fine-tuned on ORCA, PLATYPUS or others.

It is impossible to summarize what happens there. But one reference I can mention is [TheBloke](https://huggingface.co/TheBloke). 
This person focus on making available all quantizations of open-source models.
This enables everyone to run large models on commodity hardware like your laptop.

#### Oogabooga

Another parallel with the SD community, having an environment to easily run your model made everything easier.

To run stable diffusion, the reference is [Automatic1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui), named after its author. It reached 115k+ stars on Github.
The tool enabled the access to technology to a large audience. With more people, more dataset could be collected, more experiments on model could be learned, etc.

Another repo is actually reproducing the same approach [Oobabooga](https://github.com/oobabooga/text-generation-webui). It is still behind in term of maturity but also has an active community (30k stars).

This UI allows you to run any desired model with quantization, cpu or gpu, update prompt templates and even finetune a model.


![oobabooga_meme]({{site.baseurl}}/assets/oobabooga_meme.png)

#### Additional resources

[When to finetune LLM](https://medium.com/@dave-shap/a-pros-guide-to-finetuning-llms-c6eb570001d3)



## Wrap-up

![the future is now]({{site.baseurl}}/assets/future_is_now.png)

You want to know what will come next in the LLM world. Have a look in the Open Source world.
And you will need time to ingest everything, but it's best to start now.