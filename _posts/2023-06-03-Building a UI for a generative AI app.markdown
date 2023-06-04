---
description: Using Streamlit
tags: python LLM GenerativeAI  ffmpeg
img: landscape_machinery_2.png
comments: true
---


## Introduction

Building application with the latest Generative AI capabilities is a very trendy topic.

However complex applications of these technologies remain rare, and return on experience are even rarer.

In order to have stronger opinions on how to develop them, I wanted to develop a use case.

So here is the story of the creation of an app to build video stories based on Stable diffusion.


#### TLDR : 

__Character and style consistent video story creation with a Stable diffusion backend__

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238695358660955418" data-video-id="7238695358660955418" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The myth of the four suns <a title="aztec" target="_blank" href="https://www.tiktok.com/tag/aztec?refer=embed">#aztec</a>  <a title="myth" target="_blank" href="https://www.tiktok.com/tag/myth?refer=embed">#myth</a>  <a title="god" target="_blank" href="https://www.tiktok.com/tag/god?refer=embed">#god</a>  <a title="story" target="_blank" href="https://www.tiktok.com/tag/story?refer=embed">#story</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238695680183683867?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>

If you want to know how I easily build these videos, read the rest ;)




# 1 - Product evolution

The goal of the app would be to create small video stories based on a text.

The UI should help to make the creation faster, but not necessarily make everything possible.


A diagram of the logic was described in the previous post :

![Diagram]({{site.baseurl}}/assets/img/auto_gen.png){: width="900" }



## 1 - a) The brute force approach

Idea 1 : 

- A script is transformed into N scenes
- For each scene, create K images with a txt2img model
- Let it run for X hours
- The UI kicks in and the user select the best images


Why this approach : 

- Depending on your hardware, you can wait a rather long time to get all the frames necessary generated
- Also, txt2img technology is not perfect, there is sometimes obvious generation artifact that can ruin an image value (see example)



#### Some example visuals

A vision of the interface, when image has been selected for a given frame.

![Visuals of the interface 1]({{site.baseurl}}/assets/img/frame_selector_2.png){: width="700"}


Once an image is selected, it appears as bigger.

![Visuals of the interface 2]({{site.baseurl}}/assets/img/frame_selector_1.png){: width="700"}


#### Pros and cons


**Benefits** : 
- Run the logic for N scripts
- Let it run for the night
- Select the right frames to create your videos


**Drawback** : 
- You can't design each prompt manually because of the automation
- Very resource hungry
- You can still end up with no correct picture to select given a prompt



#### A focus on the failure modes : 


![The hand problem]({{site.baseurl}}/assets/img/multiple_arms.png){: width="500"}

An example of issue of the model : too many arms and glasses



## 1 - b) The creative interface

After working with the first approach for some weeks, I realised its limitations : All stories started to look the same.

As I was using ChatGPT to suggest both the content and the prompts, I was limited by what it could generate.

So I wanted to add my touch on top of the input of the system


Idea 2 : 
- Same structure as Idea 1
- I can edit each prompt if i'm unhappy with it
- Once done, I click a "generate" button, and it produces the video



#### What the UI looks like

You can see an example of the blocks that consitute a story. They can be modified to produce the wanted

![Visuals of the interface v2 1]({{site.baseurl}}/assets/img/brick_of_story_generation.png){: width="700"}

Everything can be combined together when all the frames are satisfactory for the user.

![Visuals of the interface v2 2]({{site.baseurl}}/assets/img/video_generation_in_UI.png){: width="700"}



#### Pros and cons

**Benefits** : 
- Fine grained image generation is possible


**Drawbacks** : 
- Very time consuming
- Start to get into UI complexity


#### Example of video generated

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7233298116349283610" data-video-id="7233298116349283610" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The original little Mermaid story <a title="mermaid" target="_blank" href="https://www.tiktok.com/tag/mermaid?refer=embed">#mermaid</a> <a title="disney" target="_blank" href="https://www.tiktok.com/tag/disney?refer=embed">#disney</a> <a title="tale" target="_blank" href="https://www.tiktok.com/tag/tale?refer=embed">#tale</a> <a title="sadstory" target="_blank" href="https://www.tiktok.com/tag/sadstory?refer=embed">#sadstory</a> <a title="love" target="_blank" href="https://www.tiktok.com/tag/love?refer=embed">#love</a> <a title="learn" target="_blank" href="https://www.tiktok.com/tag/learn?refer=embed">#learn</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7233299394329283354?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>



## 1 - c) Generation helper UI


The previous UI change was beneficial to the overall quality of the videos, but much more time consuming.

So, I looked at how I could add automation into the generation process.


Idea 3 - Story understanding and character consistency UI

- Get rid of ChatGPT for telling me what to put in the prompts
- Understand character references across the story with NLP
- Attach the right character description when a character is described in the scene



#### What the UI looks like


A first UI allows to describe character identified by the NLP logic 

![Visuals of the interface v3 1]({{site.baseurl}}/assets/img/character_control_1.png){: width="700"}

Multiple character can be defined

![Visuals of the interface v3 1 bis]({{site.baseurl}}/assets/img/character_control_2.png){: width="700"}


We can then come back to the original UI, prompts will have the right character description magically filled


![Visuals of the interface v3 2]({{site.baseurl}}/assets/img/character_consistency_in_story.png){: width="700"}



#### Pros and Cons

**Benefits** : 
- No need to repeat always the same prompt
- I can visualize how the character will be rendered across the story
- A character consistency technic can be used


**Drawbacks** : 
- Higher codebase complexity



#### An example video


<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@truebookwisdom/video/7238700571421642010" data-video-id="7238700571421642010" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@truebookwisdom" href="https://www.tiktok.com/@truebookwisdom?refer=embed">@truebookwisdom</a> The tale of Siegfried - Part2 <a title="knight" target="_blank" href="https://www.tiktok.com/tag/knight?refer=embed">#knight</a> <a title="tale" target="_blank" href="https://www.tiktok.com/tag/tale?refer=embed">#tale</a> <a title="dragon" target="_blank" href="https://www.tiktok.com/tag/dragon?refer=embed">#dragon</a> <a target="_blank" title="♬ original sound  - truebookwisdom" href="https://www.tiktok.com/music/original-sound-truebookwisdom-7238700853962558234?refer=embed">♬ original sound  - truebookwisdom</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>


In that video, 2 characters were used : 
- Siegfried : identified by grey hair, a certain face and an armor
- The dragon : only a simple prompt was used for it : "dark dragon"



# 2 - The UI and Tech design

A lot fo different bricks were mixed in order to get the results presented in the previous section.


## 2 - a) Working with Streamlit

Streamlit is a huge productivity boost when you don't know enough about front-end.

I learned it a bit more in order to have an efficient UI


#### The essentials to use Streamlit efficiently

**All UI widgets have a built-in shortcut**

The key parameter is what you must use

```python
import streamlit as st  

script = st.text_area("Script",
                      key="story_container",
                      value="")

if "story_container" in st.session_state:
    print(st.session_state["story_container"])
```

No need for complex callback code in order to retrieve the content of the textbox.


**Time consumming operations should be cached**

You have two operators that can be used : `st.cache_data` for long function calls and `st.cache_resource` for models.


**Use higher-order functions for callback**

When using streamlit, you will quickly want to add function calls to UI actions.

Like for this example.


```python
#prompts = list(...)
for i in range(len(prompts)):
    st.text_area(prompts[i], key=f"prompt_{i}", on_change=lambda x: image_gen(i))
```

The `i` value won't be captured for all text_area creation. Only the last value will appear.

The correct way is to use a higher order function, like in the following example.


```python
def build_gen(output_dir, index):
    prompt_key = f"prompt_{index}"
    image_key = f"image_{index}"

    def image_gen():
        prompt = st.session_state[prompt_key]
        images = api.txt2img(prompt=f"{prompt}",
                             batch_size=1, width=768, height=768).images
        st.session_state[image_key] = images[0]

    return image_gen
```

By combining everything together, we get : 

```python
#prompts = list(...)
for i in range(len(prompts)):
    fn = build_gen(output_dir, i)
    st.text_area(prompts[i], key=f"prompt_{i}", on_change=fn)
    if f"prompt_{i}" in st.session_state:
        st.image(key=f"image_{i}")
```


#### Additional UI tricks


**Save everything in states**

On UI refresh, you might end up losing information about computed states. This can be a disaster if you spent 15 minutes, tuning and updating your app content.

Let's see an example of how to make your app more stateful, given the first example seen.


```python

if "story_container" in st.session_state:
    script = st.session_state["story_container"]
else:
    script = "Hello"

script = st.text_area("Script",
                      key="story_container",
                      value=script)
```

For multiple pages app, this can allow to transfer the infors.


**Save part of the state in config**

While developping, you will often encounter situation where you broke part of your app.

If some ops are time consuming, I would advise to keep the part of state in a config file.

```python
for ... in results:
    config.append((i, speaker, who, n_speech, f"{output_dir}/image_{i}.jpg", f"{output_dir}/audio_{i}.wav"))
json.dump(config, open("...", "w"))
```

What do I save in this case.
I may have changed the speak or the content of the speak for specific part of the dialogue. Thus I avoid doing the recomputation if I get an error somewhere.



## 2 - b) Tricks around Stable Diffusion


#### Stable diffusion webui api

The [stable diffusion web ui project](https://github.com/AUTOMATIC1111/stable-diffusion-webui) offers a backend in order to control everything about the Stable diffusion model.

This was a key element in order to easily control things like models selection and generation parameter through a WebClient.

```python
api = webuiapi.WebUIApi(host='127.0.0.1', port=7860)
options = dict()
options['sd_model_checkpoint'] 
api.set_options(options)
images = api.txt2img(prompt=self.prompt, negative_prompt="bad quality",
                                 batch_size=N_GENERATION_ROUNDS).images
```

Using the [SD web ui api](https://github.com/AUTOMATIC1111/stable-diffusion-webui) offers more than a wrapper. A lot of prompt helpers are also available.

For a prompt like this one,
```
(high quality), landscape, <lora:add_detail:1>
```
we have the `()` operator helping to have a focus on a characteristic and the [Lora loader](https://github.com/microsoft/LoRA) that enable using finetuning to be specific results without changing the base model.

This could be used to have a character consistency with a trained Lora.



#### Character consistency

Character consistency is not really a feature implemented in the base Stable diffusion model.

However, celebrities are generated with their identity preserved.

This capability can be abused to have consitent character without having celebrities squatting you generations : 

```
[Robin Johansen | Rosalie Perkov] as a 31 year old male , fit, blue, outfit, smiling,  1man
```

The main idea is to generate a mix of two celebrities and force the model to generate it as the opposite sex to remove our ability to recognize the new character.

**How is it accomplished ?**

`[x | y]` is another capability of the api, where the guidance of the diffusion process alternates between `x` and `y` at each step.

The other element is to have a database of ccelebrities with their associated gender. 
Something rellatively simple to get with a db like [LFW](http://vis-www.cs.umass.edu/lfw/) or [CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html)


####  Using larger model like DeepFloyd IF

Given the limitations seen in sections 1, it is interesting to ask our selves if some model could perform better.

Today, there is mainly 3 models possible : 

- Stable diffusion 1.5 : very popular because it can be easily finetuned
- Stable diffusion 2.1 : high reolution by default, but came out later, so less support from community
- DeepFloyd IF : good at text, respect the prompt, **BUT** you need a 24gb GPU to run the full pipeline

This is the main issue with the latest model : no community support.

Even if the model is intrasically better, it wasn't finetune on quality data and looks overall less good than the much simpler SD 1.5.


