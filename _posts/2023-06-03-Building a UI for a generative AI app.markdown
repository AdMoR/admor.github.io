---
description: Using Streamlit
tags: python LLM GenerativeAI  ffmpeg
img: landscape_machinery.png
comments: true
---


## Introduction

Building application with the latest Generative AI capabilities is a very trendy topic.

However complex applications of these technologies remain rare, and return on experience are as well.

In order to have stronger opinions on how to develop them, I wanted to develop a use case.

So here is the story of the creation of an app to build video stories based on Stable diffusion.


#### TLDR : 

What the app can do : 

__Character and style consistent video story creation with a Stable diffusion backend__

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238695358660955418" data-video-id="7238695358660955418" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The myth of the four suns <a title="aztec" target="_blank" href="https://www.tiktok.com/tag/aztec?refer=embed">#aztec</a>  <a title="myth" target="_blank" href="https://www.tiktok.com/tag/myth?refer=embed">#myth</a>  <a title="god" target="_blank" href="https://www.tiktok.com/tag/god?refer=embed">#god</a>  <a title="story" target="_blank" href="https://www.tiktok.com/tag/story?refer=embed">#story</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238695680183683867?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>



## 1 - Product evolution

The goal of the app would be to create small video stories based on a text.

The UI should help to make the creation faster, but not necessarily make everything possible.


#### 1 - a) The brute force approach

Idea 1 : 

- A script is transformed into N scenes
- For each scene, create K images with a txt2img model
- Let it run for X hours
- The UI kicks in and the user select the best images


Why this approach : 

- Depending on your hardware, you can wait a rather long time to get all the frames necessary generated
- Also, txt2img technology is not perfect, there is sometimes obvious generation artifact that can ruin an image value (see example)



Some example visuals


![Visuals of the interface 1]({{site.baseurl}}/assets/img/frame_selector_2.png){: width="900"}


![Visuals of the interface 2]({{site.baseurl}}/assets/img/frame_selector_1.png){: width="900"}


![Diagram of the system 1](){}



Benefits : 
- Run the logic for N scripts
- Let it run for the night
- Select the right frames to create your videos


Drawback : 
- You can't design each prompt manually because of the automation
- Very resource hungry
- You can still end up with no correct picture to select given a prompt



Some further examples : 


![The hand problem]({{site.baseurl}}/assets/img/multiple_arms.png){}




#### 1 - b) The creative interface

After working with the first approach for some weeks, I realised its limitations : All stories started to look the same.

As I was using ChatGPT to suggest both the content and the prompts, I was limited by what it could generate.

So I wanted to add my touch on top of the input of the system


Idea 2 : 
- Same structure as Idea 1
- I can edit each prompt if i'm unhappy with it
- Once done, I click a "generate" button, and it produces the video



Some example visuals


![Visuals of the interface v2 1]({{site.baseurl}}/assets/img/brick_of_story_generation.png){: width="900"}


![Visuals of the interface v2 2]({{site.baseurl}}/assets/img/video_generation_in_UI.png){: width="900"}


![Diagram of the system 2](){}



Benefits : 
- Fine grained image generation is possible


Drawback : 
- Very time consuming
- Start to get into UI complexity


**Example of video generated**

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7233298116349283610" data-video-id="7233298116349283610" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The original little Mermaid story <a title="mermaid" target="_blank" href="https://www.tiktok.com/tag/mermaid?refer=embed">#mermaid</a> <a title="disney" target="_blank" href="https://www.tiktok.com/tag/disney?refer=embed">#disney</a> <a title="tale" target="_blank" href="https://www.tiktok.com/tag/tale?refer=embed">#tale</a> <a title="sadstory" target="_blank" href="https://www.tiktok.com/tag/sadstory?refer=embed">#sadstory</a> <a title="love" target="_blank" href="https://www.tiktok.com/tag/love?refer=embed">#love</a> <a title="learn" target="_blank" href="https://www.tiktok.com/tag/learn?refer=embed">#learn</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7233299394329283354?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>


#### 1 - c) Generation helper UI


The previous UI change was beneficial to the overall quality of the videos, but much more time consuming.

So, I looked at how I could add automation into the generation process.


Idea 3 - Story understanding and character consistency UI

- Get rid of ChatGPT for telling me what to put in the prompts
- Understand character references across the story with NLP
- Attach the right character description when a character is described in the scene



**Let's see how this works**


![Visuals of the interface v3 1]({{site.baseurl}}/assets/img/character_control_1.png){: width="900"}
![Visuals of the interface v3 1 bis]({{site.baseurl}}/assets/img/character_control_2.png){: width="900"}

A first UI allows to describe character identified by the NLP logic 



![Visuals of the interface v3 2]({{site.baseurl}}/assets/img/character_consistency_in_story.png){: width="900"}

We can then come back to the original UI and prompt will have the right ccharacter description magically filled



**Pros and Cons**

Benefits : 
- No need to repeat always the same prompt
- I can visualize how the character will be rendered across the story
- A character consistency technic can be used


Drawback : 
- Higher codebase complexity


**An example video**


<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238700571421642010" data-video-id="7238700571421642010" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The tale of Siegfried - Part2 <a title="knight" target="_blank" href="https://www.tiktok.com/tag/knight?refer=embed">#knight</a> <a title="tale" target="_blank" href="https://www.tiktok.com/tag/tale?refer=embed">#tale</a> <a title="dragon" target="_blank" href="https://www.tiktok.com/tag/dragon?refer=embed">#dragon</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238700853962558234?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>


In that video, 2 characters were used : 
- Siegfried : identified by grey hair, a certain face and an armor
- The dragon : only a simple prompt was used for it : "dark dragon"



## 2 - The UI and Tech design


#### 2 - a) Working with Streamlit




#### 2 - b) Tricks around Stable Diffusion


**Stable diffusion webui api**



**Character consistency**



**Using larger model like Floyd IT**



