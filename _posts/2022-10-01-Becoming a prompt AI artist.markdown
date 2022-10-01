---
description: A practical non technical guide
tags: stable diffusion sd python generative
img: style_transfer_sd.jpeg
comments: true
---


## Introduction

In this post, we are going to see what we can do with stable diffusion and how to get faster to satisfying results.


## The tools

In order to be a prompt artist, you need the following ressources : 

- A gpu installed on your computer or cloud instance (I have a GTX 1070)
- [Low memory stable diffusion repo](https://github.com/basujindal/stable-diffusion)
- The [prompt book](https://dallery.gallery/the-dalle-2-prompt-book/) which describe how to reach certain visualization
- [optional] [An account on Dalle2](https://labs.openai.com/)



## The gallery

Some example of generations with their original prompt.



![result_1]({{site.baseurl}}/assets/img/machine_to_steam_corn.png){: width="750" }



![result_2]({{site.baseurl}}/assets/img/armored_developer.png){: width="750" }



![result_3]({{site.baseurl}}/assets/img/eldritch_horror.png){: width="750" }


## Focusing on cross overs

A lot has already been discussed and done on the subject of generation with stable diffusion 


#### Style transfer

```
A_car_looking_like_Sonic_the_hedgehog,_3D_render,_trending_on_artstation
```

![result_5]({{site.baseurl}}/assets/img/sonic_car_2.png){: width="750" }

A good example of the transfer of "Sonic features" to a car.



![result_6]({{site.baseurl}}/assets/img/sonic_car.png){: width="750" }

Here I believe the car term as be understood as Car from Pixar. And the result is great



Another example of style transfer can be achieved with a predefined image. 


![result 7]({{site.baseurl}}/assets/img/style_transfer_sd.jpeg){: width="750" }

Here a real life image is adapted to fit respect the impressionist style.



#### Style transfer across brands

```
Thomas_the_train_engine,_in_GTA_V,_cover_art_by_Stephen_Bliss,_artstation
```

![result_1]({{site.baseurl}}/assets/img/thomas_the_train_gta4.png){: width="750" }


![result_2]({{site.baseurl}}/assets/img/Thomas_train.png){: width="750" }


At first, I had the impression of the GTA4 not working properly. So i tried another one.


```
Kermit_the_frog,_in_GTA_V,_cover_art_by_Stephen_Bliss,_artstation
```


![result_3]({{site.baseurl}}/assets/img/kermit_cover_gta4.png){: width="750" }


![result_4]({{site.baseurl}}/assets/img/kermit_character_gta4.png){: width="750" }


Here we recognize the style of the game.

However when looking more precisely at the background, we can better understand the background of Thomas the train.

From there, it's clear that the stable diffusion model needs to tick some boxes and desert background might tick GTA4.


```
Kermit_the_frog_as_a_World_of_warcraft_character,_simplistic_3D_render
```

Here we are looking for additional support from the last point. The next example seems to support that.


![result_4.b]({{site.baseurl}}/assets/img/kermit_in_wow.png){: width="750" }



## Using outpainting for better results

I want to generate a children book cover.

After several try, i find an image that I like

![result_4.b]({{site.baseurl}}/assets/img/derp_the_chicken.png){: width="750" }


In the open ai interface, I have a handy way of removing what I don't want.

The final result has its main imperfections removed.

![result_4.b]({{site.baseurl}}/assets/img/dep_the_chicken_v2.png){: width="750" }


## Conclusion

I hope you will have as much fun as I did.

