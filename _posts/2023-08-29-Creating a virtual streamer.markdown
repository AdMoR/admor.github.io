---
description: on twitch
tags: python LLM GenerativeAI  ffmpeg
img: AIJesusFace.jpg
comments: true
---


## Introduction

The goal of this post is to reveal how I built a virtual streamer capable of : 

- interacting with a chat 
- respond with text to speech
- animate a video and more precisely the lips of the character


## Too Long Didn't Read : 

An example of a response to a question.

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7272815988502957345" data-video-id="7272815988502957345" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> <a title="jesus" target="_blank" href="https://www.tiktok.com/tag/jesus?refer=embed">#Jesus</a> parle des contradictions de l&#39;homme <a title="fr" target="_blank" href="https://www.tiktok.com/tag/fr?refer=embed">#fr</a> <a title="learn" target="_blank" href="https://www.tiktok.com/tag/learn?refer=embed">#learn</a> <a title="ai" target="_blank" href="https://www.tiktok.com/tag/ai?refer=embed">#AI</a> <a title="ia" target="_blank" href="https://www.tiktok.com/tag/ia?refer=embed">#IA</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7272815988357286688?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>

Ask your question on [Twitch](https://www.twitch.tv/allojesuschrist)




## General overview

The system can be summarized with the following diagram : 


![architecture diagram]({{site.baseurl}}/assets/img/AIJesusDiagram.png)


The system is composed of 3 parts : 
- The ChatReader. It reads the twitch chat, if new message arrives they are added in a high prio queue.
- The VideoBuilder. It builds the vidoe response to the question, it is composed of 3 steps that we will detail later.
- The Streamer. It takes cares of streaming videos to twitch, if new videos arrives, they are read first.



## Overview of the VideoBuilder

The VideoBuilder is composed of the main steps of the go from a question in text format to a video answering the question.


I used : 
- ChatGPT to anwswer user questions
- TTS to convert the response to an audio format
- VideoRetalking to use an input video and input audio and create a lip synced video.


## Making it efficient

As I was hosting everything on my desktop computer, I had to tweak it to make more reactive.

The system was originally very slow and I had to do a few tweaks to the VideoRetalking code.

The consequence is the obvious video artefacts visible on the output.

But it allowed to reach a half real time speed with an RTX 4080.


## Next steps 

I would like to test : 

- different prompts to reach more interesting answers
- make the code even faster to reach real time
- improve the voice quality - much lower in french compared to available english ones
- and make the whole system more production ready

