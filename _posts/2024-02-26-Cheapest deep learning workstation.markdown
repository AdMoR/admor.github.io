---
description: Challenge : with only 7 days of EC2 GPU compute
tags: GPU cloud deep learning
img: p40.png
comments: true
---


## Introduction


Let's suppose that you are a deep learning enthusiast. 

You want to test a new fancy open source model like speech2text, text2speech, txt2img or event txt2video. 

What should you do ? 
- Pay for a platform to do your tests
- Rent a cloud computer
- Use a colab notebook
- Run it on your own hardware


I'm personally in favor of the latter, as you learn more.
But the main challenge is the **cost** of the hardware.


In this blog post, I will reveal my secret to build the cheapest deep learning workstation you have ever seen.


## The formula

The formula is simple : buy **used parts** on Ebay or LeBonCoin.

I can offer several declinations 


### Cheapest formula

40€ motherboard + CPU + RAM combo

![A used motherboard with cpu]({{site.baseurl}}/assets/img/board_32gb.png){: width="550" }


9€ power supply 

![Power supply]({{site.baseurl}}/assets/img/cheap_power_supply.png){: width="550" }


96€ K80 GPU

![GPU from older times]({{site.baseurl}}/assets/img/k80.png){: width="550" }


#### Budget deep learning workstation

**145€** for our budget setup

vs 

**168€** for 7 days of full time [g5.xl](https://aws.amazon.com/ec2/instance-types/g5/) time 



### SCALING formaula


Upgrade the GPU to a P40 or multiple


170€ P40 GPU

![GPU from older times 2]({{site.baseurl}}/assets/img/p40.png){: width="550" }

Because it is seen as a single 24Gb gpu and not multiple

With multiple GPU, you can approach the price of more expansive instance like the [g5.12xlarge]


#### Scaling deep learning workstation

110€ motherboard + CPU + 128gb RAM combo

![A used motherboard with cpu, v2]({{site.baseurl}}/assets/img/better_board.png){: width="550" }


46€ 1300w power supply 

![Power supply]({{site.baseurl}}/assets/img/powerful_power_supply.png){: width="550" }


410€ : 3 x P40 GPU

![GPU from older times 2]({{site.baseurl}}/assets/img/p40.png){: width="550" }


Price : 560€



#### Reference 

7 days of g5.12xlarge = 953€
We assume we need only 3 GPUs = 715€


So 22% cheaper than AWS for a 3 GPU instance


![What it could look like](https://preview.redd.it/nvidia-tesla-p40-performs-amazingly-well-for-gguf-v0-1d0t9kf5ji1c1.jpg?width=4032&format=pjpg&auto=webp&s=41dc5adba846fc6f23e603127be4054cab789206)



## The catch

These numbers are unfortunately hidding a few things : 

- Power consumption : it will be hot in the room hosting your new machine
- Lower performance : the GPU are a bit outdated, inference performance will be below the professional cards
- Training : you may consider another setup if you want to train a large model, as it will be too slow and energy hungry

But.... 

We compared to a g5 instance, which packs a A10 GPU, very far from state of the art

- The A100 is a reference for enterprise grade deep learning (but is extremely costly)
- On some [benchmarks](https://www.baseten.co/blog/nvidia-a10-vs-a100-gpus-for-llm-and-stable-diffusion-inference/), the A10 is **2x slower** than the A100 (on stable diffusion) 
- But the [A10G is still 50% above the P40](https://technical.city/en/video/Tesla-P40-vs-A10G)
- But when comparing with a 500€ consumer GPU card, the [A10G would be 30% below the 4060 ti](https://technical.city/en/video/A10G-vs-GeForce-RTX-4060-Ti) which is 500€ card


All in all, renting a cloud GPU is not a very good deal if you are not a company.


## Appendix 

The sources I used to build this article : 

- The local llama sub-reddit : https://www.reddit.com/r/LocalLLaMA/
- [A comparaison of the different GPU options](https://www.reddit.com/media?url=https%3A%2F%2Fpreview.redd.it%2Fthe-llm-gpu-buying-guide-august-2023-v0-4nve5pq5oaib1.png%3Fwidth%3D1248%26format%3Dpng%26auto%3Dwebp%26s%3Dd6c59b7fdc75f671d933299b581c800d5adbb6ef)







