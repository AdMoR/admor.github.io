---
description: Using Streamlit
tags: python LLM GenerativeAI  ffmpeg
img: coreference_clusters.gif
comments: true
---


## Introduction

In [this previous post](https://admor.github.io/Infinite-motivational-video-creation-with-ChatGPT/), I presented how to generate videos based on stable diffusion.

In this post, I wanted to present the UI used to simply build the videos, and the challenges met while building it. 


## Too Long Didn't Read : 

An example of the video story generated.

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238695358660955418" data-video-id="7238695358660955418" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The myth of the four suns <a title="aztec" target="_blank" href="https://www.tiktok.com/tag/aztec?refer=embed">#aztec</a>  <a title="myth" target="_blank" href="https://www.tiktok.com/tag/myth?refer=embed">#myth</a>  <a title="god" target="_blank" href="https://www.tiktok.com/tag/god?refer=embed">#god</a>  <a title="story" target="_blank" href="https://www.tiktok.com/tag/story?refer=embed">#story</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238695680183683867?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>

If you want to know how I build these videos, read the rest ;)




## General overview

The goal of the app would be to create small video stories based on a text.

The UI should help to make the creation faster, but not necessarily make everything possible.


A diagram of the logic was described in [the previous post](https://admor.github.io/Infinite-motivational-video-creation-with-ChatGPT/) :

![Diagram]({{site.baseurl}}/assets/img/auto_gen.png){: width="700" }

In short, ChatGPT generates a story. Each sentence is translated to an image with Stable Diffusion. And finally txt2speech adds the narration voice.



## Idea 1 : the brute force approach


#### > The overview of the logic

- We start with a large block of text that we will split in 10 sentences.
- Each sentence will be represented by 1 image
- For each sentence, create 15 images to choose from (with a txt2img model like Stable Diffusion)
- A UI allows me to pick the best image for each scene
- Finally all images and audio are combined into the video



#### > Let's look at the UI


![Video of the interface]({{site.baseurl}}/assets/img/bruteforce_interface.gif)

Once the generation process is finished, we only need to select good images corresponding to the story.



#### > Why this approach : 

- Depending on your hardware, you can wait a rather long time to get 15 images generated for a prompt
- Also, txt2img technology is not perfect, there is sometimes obvious generation artifact that can ruin an image value (see example)




#### > A focus on the failure modes : why we need more than 1 image per sentence

Often SD1.5 models struggle on small details like hands, arms or even ... glasses. 

![The hand problem]({{site.baseurl}}/assets/img/multiple_arms.png){: width="300"}

Here the count is not right.



## Idea 2 : the creative interface

After working with the first approach for some weeks, I realised its limitations : all stories started to look the same and needed some manual editing.

As I was using ChatGPT to suggest both the content and the prompts, I was limited by what it could generate.

Additional controls were thus needed in the UI.


#### > Overview of the logic : 
- Same structure as Idea 1
- I can edit each prompt if i'm unhappy with it
- Once done, I click a "generate" button, and it produces the video



![Video of the interface]({{site.baseurl}}/assets/img/text_parsing_to_video.gif)

In this video, images have already been generated based on the input story. And only one frame needs a change in the prompt. Finally, the video is finally combined with the final generate button.



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


#### > Overview of the logic : 

- We have the input story which is a large block of text
- I use a model from Spacy to detect multiple references of the same character in this story
- Characters are represented by a cluster of words
- Each cluster will get its own prompt
- When an element of a cluster is found in a sentence, we add all the prompt words attached to this cluster
- This mechanism helps to automatically fill the prompts for each sentence



![Video of the interface]({{site.baseurl}}/assets/img/coreference_clusters.gif)

In this example, two characters are identified and assigned a prompt to get a consistent visual identity. 


#### > Focus on the UI


The UI achieves different purposes : 

- Manually identify what is cluster is about with the words of the cluster
- Define the prompt words for the cluster
- Preview how a prompt will consistently generate a character


![Visuals of the interface v3 1]({{site.baseurl}}/assets/img/character_control_2.png){: width="550"}
![Visuals of the interface v3 1 bis]({{site.baseurl}}/assets/img/character_control_1.png){: width="550"}


Back to the original UI, prompts will have the right character description magically filled


![Visuals of the interface v3 2]({{site.baseurl}}/assets/img/character_consistency_in_story.png){: width="550"}




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