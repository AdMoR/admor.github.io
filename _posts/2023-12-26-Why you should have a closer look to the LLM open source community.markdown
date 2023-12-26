---
description: and where to look at
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

![knuckles.png]({{site.baseurl}}/assets/img/ugandan_knuckles.png){: width="700" }

With the more recent releases of Stable-diffusion-XL, the gap between proprietary and open-source models remains limited.

![knuckles.png]({{site.baseurl}}/assets/img/midjou_vs_sdxl.png){: width="700" }

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

![knuckles.png]({{site.baseurl}}/assets/img/localllama.png){: width="700" }

This subreddit focuses on harnessing the open source LLM and making.
And things can get technical really quickly, even for machine learning practitioners.

![knuckles.png]({{site.baseurl}}/assets/img/is_this_satire.png){: width="700" }

I assure you this is not satire and an actual feedback from someone who tested many models.

What to expect from this subreddit ?
- Practitioner advices on RAG and finetuning best practices
- Benchmarks to compare models
- Tests in production performance

**Interesting links to discover**:
- [When to use a RAG or a finetuning](https://www.reddit.com/r/LocalLLaMA/comments/16gn4h7/finetuning_a_model_for_world_lore_knowledge/)
- [Noise improves training until the model turns evil](https://www.reddit.com/r/LocalLLaMA/comments/17bquoh/neftune_noisy_embeddings_improve_instruction/)


#### Huggingface model repository

A lot of people are experimenting with models thanks to huggingface. 
But isolated experiments wouldn't bring as much as the model repository that Huggingface is today.

If we come back to the parallel of txt-2-img, finetuning on custom dataset have been used to learn how to generate a better base model.
You could find pretty advanced discussions on the model merging topic on reddit [1](https://www.reddit.com/r/StableDiffusion/comments/11sqsk8/merge_models_by_layers_and_similarity/), [2](https://www.reddit.com/r/StableDiffusion/comments/18ov9cg/comparison_opendalle_sdxl_model_merged_with_dpo/)

You can find a similar trend on HuggingFace for the LLM. Mistral fine-tuned on ORCA, PLATYPUS or others.

It is impossible to summarize what happens there. But one reference I can mention is [TheBloke](https://huggingface.co/TheBloke). 
This person focus on making available all quantizations of open-source models.
This enables everyone to run large models on commodity hardware like your laptop.

**Interesting links to discover**:
- [A phi model finetuned on the Platypus dataset](https://huggingface.co/SkunkworksAI/PlatyPhi-1.5B?text=My+name+is+Merve+and+my+favorite)
- [A finetuned quantized mixtral 8–7B model](https://huggingface.co/TheBloke/dolphin-2.5-mixtral-8x7b-GGUF)


#### Oogabooga

Another parallel with the SD community, having an environment to easily run your model made everything easier.

To run stable diffusion, the reference is [Automatic1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui), named after its author. It reached 115k+ stars on Github.
The tool enabled the access to technology to a large audience. With more people, more dataset could be collected, more experiments on model could be learned, etc.

Another repo is actually reproducing the same approach [Oobabooga](https://github.com/oobabooga/text-generation-webui). It is still behind in term of maturity but also has an active community (30k stars).

This UI allows you to run any desired model with quantization, cpu or gpu, update prompt templates and even finetune a model.


![oobabooga_meme]({{site.baseurl}}/assets/img/oobabooga_meme.png){: width="700" }

**Interesting links to discover**:
- [Oobabooga subreddit](https://www.reddit.com/r/Oobabooga/top/?t=month)

    


#### Additional resources

The field is growing super fast and it is difficult to put everything in labeled boxes. 

For those interested by specific topics like finetuning, I added a list of useful resources

[When to finetune LLM](https://medium.com/@dave-shap/a-pros-guide-to-finetuning-llms-c6eb570001d3)



## Wrap-up

![the future is now]({{site.baseurl}}/assets/img/future_is_now.png)

You want to know what will come next in the LLM world. 

Have a look in the Open Source world. And you will need time to ingest everything, but it's best to start now.