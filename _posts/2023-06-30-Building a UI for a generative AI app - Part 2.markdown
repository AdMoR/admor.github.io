---
description: The technical tricks from part 1
tags: python LLM GenerativeAI  ffmpeg
img: landscape_machinery_2.png
comments: true
---


## Introduction

This post focuses on the technical elements of the video creation based on txt2img technologies



# The UI and Tech design

A lot fo different bricks were mixed in order to get the results presented in the previous section.


## a) Working with Streamlit

Streamlit is a huge productivity boost when you don't know enough about front-end.

I learned it a bit more in order to have an efficient UI


#### >> The essentials to use Streamlit efficiently

**> All UI widgets have a built-in shortcut**

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


**> Time consumming operations should be cached**

You have two operators that can be used : `st.cache_data` for long function calls and `st.cache_resource` for models.


**> Use higher-order functions for callback**

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


#### >> Additional UI tricks


**> Save everything in states**

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


**> Save part of the state in config**

While developping, you will often encounter situation where you broke part of your app.

If some ops are time consuming, I would advise to keep the part of state in a config file.

```python
for ... in results:
    config.append((i, speaker, who, n_speech, f"{output_dir}/image_{i}.jpg", f"{output_dir}/audio_{i}.wav"))
json.dump(config, open("...", "w"))
```

What do I save in this case.
I may have changed the speak or the content of the speak for specific part of the dialogue. Thus I avoid doing the recomputation if I get an error somewhere.



## b) Tricks around Stable Diffusion


#### >> Stable diffusion webui api

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



#### >> Character consistency

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


####  >> Using larger model like SDXL and DeepFloyd IF

Given the limitations seen in sections 1, it is interesting to ask our selves if some model could perform better.

Today, there is mainly 3 models possible : 

- Stable diffusion 1.5 : very popular because it can be easily finetuned
- Stable diffusion 2.1 : high reolution by default, but came out later, so less support from community
- DeepFloyd IF : good at text, respect the prompt, **BUT** you need a 24gb GPU to run the full pipeline
- Stable diffusion XL : high quality, larger memory footprint and inference time, community models development ongoing

So currently, using a 1.5 model can still hold in term of quality given a compute budget. Especially given some Lora like \<add detail\> are great at improving the model with close to no cost.
    
But we can expect that a new generation of SDXL model will take over. 


