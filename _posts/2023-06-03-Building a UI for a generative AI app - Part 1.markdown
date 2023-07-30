---
description: Using Streamlit
tags: python LLM GenerativeAI  ffmpeg
img: coreference_clusters.gif
comments: true
---


## Introduction

Building application with the latest Generative AI capabilities is a very trendy topic.

However complex applications of these technologies remain rare, and return on experience are even rarer.

In order to have stronger opinions on how to develop them, I wanted to develop a use case.

So here is the story of the creation of an app to build video stories based on Stable diffusion.


## TLDR : 

__Character and style consistent video story creation with a Stable diffusion backend__

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238695358660955418" data-video-id="7238695358660955418" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The myth of the four suns <a title="aztec" target="_blank" href="https://www.tiktok.com/tag/aztec?refer=embed">#aztec</a>  <a title="myth" target="_blank" href="https://www.tiktok.com/tag/myth?refer=embed">#myth</a>  <a title="god" target="_blank" href="https://www.tiktok.com/tag/god?refer=embed">#god</a>  <a title="story" target="_blank" href="https://www.tiktok.com/tag/story?refer=embed">#story</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238695680183683867?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>

If you want to know how I easily build these videos, read the rest ;)




## General overview

The goal of the app would be to create small video stories based on a text.

The UI should help to make the creation faster, but not necessarily make everything possible.


A diagram of the logic was described in [the previous post](https://admor.github.io/Infinite-motivational-video-creation-with-ChatGPT/) :

![Diagram]({{site.baseurl}}/assets/img/auto_gen.png){: width="900" }



## Idea 1 : the brute force approach


The overview of the logic

- We start with a large block of text that we will split in 10 sentences.
- Each sentence will be represented by 1 image
- For each scene, create 15 images to choose from (with a txt2img model)
- A UI allows me to pick the best image for each scene



#### > Some example visuals



![Video of the interface]({{site.baseurl}}/assets/img/bruteforce_interface.gif)


A vision of the interface, when image has been selected for a given frame.

Once an image is selected, it appears as bigger.



#### > Why this approach : 

- Depending on your hardware, you can wait a rather long time to get all the frames necessary generated
- Also, txt2img technology is not perfect, there is sometimes obvious generation artifact that can ruin an image value (see example)




#### > A focus on the failure modes : 


![The hand problem]({{site.baseurl}}/assets/img/multiple_arms.png){: width="300"}

An example of issue of the model : too many arms and glasses



## Idea 2 : the creative interface

After working with the first approach for some weeks, I realised its limitations : all stories started to look the same.

As I was using ChatGPT to suggest both the content and the prompts, I was limited by what it could generate.

So I wanted to have more control on the generation.


Overview : 
- Same structure as Idea 1
- I can edit each prompt if i'm unhappy with it
- Once done, I click a "generate" button, and it produces the video



![Video of the interface]({{site.baseurl}}/assets/img/text_parsing_to_video.gif)




#### > The UI in details

The UI descibes how the story is broken down in smaller blocks.
Each block correspond to an image. If the prompt for this image does not provide a satisfactory image. One can change the prompt.

However we lose the ability to choose among a list of 10 images as everything is synchronous.

![Visuals of the interface v2 1]({{site.baseurl}}/assets/img/brick_of_story_generation.png){: width="550"}

It is possible to preview the final video once all the images are generated.

![Visuals of the interface v2 2]({{site.baseurl}}/assets/img/video_generation_in_UI.png){: width="550"}




#### > Example of video generated

In this example, I took the story of the little mermaid outputed by ChatGPT and tuned the prompt for better visuals.

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7233298116349283610" data-video-id="7233298116349283610" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The original little Mermaid story <a title="mermaid" target="_blank" href="https://www.tiktok.com/tag/mermaid?refer=embed">#mermaid</a> <a title="disney" target="_blank" href="https://www.tiktok.com/tag/disney?refer=embed">#disney</a> <a title="tale" target="_blank" href="https://www.tiktok.com/tag/tale?refer=embed">#tale</a> <a title="sadstory" target="_blank" href="https://www.tiktok.com/tag/sadstory?refer=embed">#sadstory</a> <a title="love" target="_blank" href="https://www.tiktok.com/tag/love?refer=embed">#love</a> <a title="learn" target="_blank" href="https://www.tiktok.com/tag/learn?refer=embed">#learn</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7233299394329283354?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>



## Idea 3 : Character generation helper UI


The previous UI change was beneficial to the overall quality of the videos, but much more time consuming.

So, I looked at how I could add automation into the generation process : by making character generation consistent 


Overview of the logic : 

- I have a large block of text
- I use a model from Spacy to detect multiple references of the same character
- Characters are represented by cluster of words
- Each cluster will get its own prompt
- When an element of a cluster is found in a sentence, we add all the prompt words attached to this cluster



![Video of the interface]({{site.baseurl}}/assets/img/coreference_clusters.gif)




#### > Focus on the UI


The UI achieves different purposes : 

- Manually identify what is cluster is about with the words of the cluster
- Define the prompt words for the cluster
- Preview how a prompt will consistently generate a character


![Visuals of the interface v3 1]({{site.baseurl}}/assets/img/character_control_2.png){: width="500"}
![Visuals of the interface v3 1 bis]({{site.baseurl}}/assets/img/character_control_1.png){: width="500"}


Back to the original UI, prompts will have the right character description magically filled


![Visuals of the interface v3 2]({{site.baseurl}}/assets/img/character_consistency_in_story.png){: width="500"}




#### > An example video


In that video, 2 characters were used : 
- Siegfried : identified by grey hair, a certain face and an armor
- The dragon : only a simple prompt was used for it : "dark dragon"


<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238700571421642010" data-video-id="7238700571421642010" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The tale of Siegfried - Part2 <a title="knight" target="_blank" href="https://www.tiktok.com/tag/knight?refer=embed">#knight</a> <a title="tale" target="_blank" href="https://www.tiktok.com/tag/tale?refer=embed">#tale</a> <a title="dragon" target="_blank" href="https://www.tiktok.com/tag/dragon?refer=embed">#dragon</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238700853962558234?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>




## Conclusion


With this tooling, I was able to turn a txt2img technology to a video creation tool.

A lot of additional development could be made like : 

- [Automatic animation](https://animatediff.github.io/)
- [Using a better model](https://stability.ai/blog/stable-diffusion-sdxl-1-announcementani)
- [Generate my own music](https://ai.honu.io/papers/musicgen/)