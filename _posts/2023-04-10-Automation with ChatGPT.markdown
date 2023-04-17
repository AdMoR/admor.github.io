---
description: An example at building a long automation chain
tags: python LLM ffmpeg
img: synergy.png
comments: true
---


## Introduction

My goal in this project was to see how I could use the new capabilities of ChatGPT.

What came clear to me and many other users, is that ChatGPT can be like a better Google.

It know about a lot of concepts : from Pokemon to Friends.

My goal : automate story building using open source components and ChatGPT for the story building


## 1 - The architecture

#### 1 - a) Prompt engineering

A prompt template is tailored to get a scenario that we can then parse 

```
Generate a top of life advices. Make Several small sentences rather than big ones.
Use this template format : after a quote, separated by a | , the visual of the scene is described.

Title : Top advices from "The art of War"
Narrator : "If you know the enemy and know yourself, you need not fear the result of a hundred battles." | A man is wielding a sword and faces the camera
Narrator : Similarly, in life. It is important to understand your strengths and weaknesses. | A person looking at his hands
Narrator :  But also those of your adversaries. | A person weakness
Narrator : This knowledge can help you to navigate challenges and make strategic decisions. | A book, a medieval helmet and a knife
 
Now, generate a set of advices from {prompt} with the format defined above : 
```

We force the geenration to add a scene description in order to accomodate our generation later on.


#### 1 - b) Text parsing

We need to retrieve : 
- The character speaking
- The scene prompt
- And the speech of the character

There are some specificities to this format. We want to keep the sentences short and energic. 
So we might want ot break down the sentences in smaller chunks.


#### 1 - c) Media generation with StableDiffusion and TTS

Each lines in our previous dataset gives one speech synthesis and one image.



For the image generation : 


```python
model_id = "stabilityai/stable-diffusion-2-1"
scheduler = EulerDiscreteScheduler.from_pretrained(model_id, subfolder="scheduler")
pipe = StableDiffusionPipeline.from_pretrained(model_id, scheduler=scheduler, torch_dtype=torch.float16)
pipe = pipe.to("cuda")
N_STEPS = 150
GUIDANCE_SCALE = 15

image = pipe(scene_prompt, num_inference_steps=N_STEPS, guidance_scale=GUIDANCE_SCALE).images[0]
```

For the TTS generation : 


```python
language = 'en'
model_id = 'v3_en'
sample_rate = 48000
speaker = 'en_1'
model, example_text = torch.hub.load(repo_or_dir='snakers4/silero-models',
                                     model='silero_tts',
                                     language=language,
                                     speaker=model_id)

model.save_wav(text=text,
               audio_path=audio_path,
               speaker=speaker,
               sample_rate=sample_rate)
```



#### 1 - d) Automation with ffmpeg

All the pieces are merged together using FFMPEG.

One example of merging audio and image : 

```python
def combine_img_audio(png_file_path, audio_path, mp4_file_path):
    rez = subprocess.run(["ffmpeg", "-y", "-i", audio_path, "-i", png_file_path,
                          "-framerate", "1", mp4_file_path])
    if rez.returncode == 1:
        raise Exception("ffmpeg audio+image failed")
    return mp4_file_path

```



## 2 - Some examples


### Experiment 1 : A pokemon episode example

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@aipokemonscripts/video/7217535697077423365" data-video-id="7217535697077423365" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@aipokemonscripts" href="https://www.tiktok.com/@aipokemonscripts?refer=embed">@aipokemonscripts</a> Trapped in the loop <a title="ai" target="_blank" href="https://www.tiktok.com/tag/ai?refer=embed">#ai</a> <a title="pokemon" target="_blank" href="https://www.tiktok.com/tag/pokemon?refer=embed">#pokemon</a> <a title="ytp" target="_blank" href="https://www.tiktok.com/tag/ytp?refer=embed">#ytp</a> <a title="youtubepoop" target="_blank" href="https://www.tiktok.com/tag/youtubepoop?refer=embed">#youtubepoop</a> <a target="_blank" title="♬ original sound - aipokemonscripts" href="https://www.tiktok.com/music/original-sound-7217535679486495494?refer=embed">♬ original sound - aipokemonscripts</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>


#### Review 

The negative

- Poor image quality 
- We expect more character focus when someone is talking
- We lose track of the speaker count


The positive
- Some rythm
- An original proposition



### Experiment 2 : An inspirational video

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7220408946287299846" data-video-id="7220408946287299846" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> Life learnings from Winning friends and Influence people <a title="coach" target="_blank" href="https://www.tiktok.com/tag/coach?refer=embed">#coach</a>  <a title="motivational" target="_blank" href="https://www.tiktok.com/tag/motivational?refer=embed">#motivational</a>  <a title="lifehack" target="_blank" href="https://www.tiktok.com/tag/lifehack?refer=embed">#lifehack</a> <a title="inspirational" target="_blank" href="https://www.tiktok.com/tag/inspirational?refer=embed">#inspirational</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7220409382361959173?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>


#### Review 

The negative
- Image content can be unexpected


The positive
- Enjoyable
- Overall close to a human production
