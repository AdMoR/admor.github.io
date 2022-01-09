---
description: Summary of a journey to create a novel kind of shape algorithm
tags: python svg neuron excitation optimization pytorch
img: axidraw_drawing.jpg
comments: true
---

## Introduction

I have for a long time looked for a way to mix image generation and the real world.
The way to bring something digital into the real world could be a printer. However one usual limitation of image generation is that it is low dimension usually 1000x1000. 

[Dithering](https://en.wikipedia.org/wiki/Dither) can be interesting but the results are not special enough.


## Differential SVG

#### Differential what ?

Before talking about the library, let's talk about the format. SVG for scalable vector graphics allows to draw a lot of stuff and unlike an image, it can be scaled to any desired size.

The main tool that will be used is [the bezier curve](https://en.wikipedia.org/wiki/Bézier_curve). It is our atomic element to represent a stroke of a pen. In order to create a drawing, we will use a lot of them.


__So about the library__ : 

The base of our size application is the library [diffvg](https://github.com/BachiLi/diffvg) that enables optimizing the parameters of a svg curve.
It uses a concept similar to [the differentiable render](https://www.youtube.com/watch?v=tGJ4tEwhgo8).


#### Learning an interesting drawing for the network

The basic idea here is similar to what is done in [this article](https://distill.pub/2018/differentiable-parameterizations/), finding the image that maximise the activation of a specific neuron in a neural network.


```
{svg_lines} ===(render)===> image ==(neural net)==> layer => loss
```

Additionally in order to have a more stable optimization process, some data augmentation are used : random crop, perspective, jitters.


The logic for the code can be found [here](https://github.com/AdMoR/neural-styles/blob/master/svg_neuron_optim.py).


#### Some results

Old school neural networks like VGG16 or ResNet18 allow to have an optimization process fast enough with interesting feature as well.


*With color :*

![VGG Conv3_3](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/best_svg_color_neuron_exc/result_n_paths202_im_size500_n_steps2500_layer_nameVGGLayers.Conv3_3_layer_index37.svg)

VGG16 Conv3_3 layer 37


![VGG Conv3_3](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/best_svg_color_neuron_exc/result_n_paths202_im_size500_n_steps2500_layer_nameVGGLayers.Conv3_3_layer_index7.svg)

VGG16 Conv3_3 layer 7


*Without :*

![larger]({{site.baseurl}}/assets/img/saxi_original_drawing.png)



## From data to drawing

#### Naive implementation 

Correctly drawing a svg with the axidraw ends up being a difficult task. Fortunately there are great tools like [saxi](https://github.com/nornagon/saxi) to facilitate the job.

One can drag and drop the file and visualize what the drawing would look like on a given sheet of paper.


Given this image 

![larger]({{site.baseurl}}/assets/img/saxi_original_drawing.png)

This is what is seen.

![larger]({{site.baseurl}}/assets/img/saxi_naive_drawing.png)

You may notice that the stroke has a different size on the original file compared to the forecasted plot. This is normal as it is a parameter of the drawing process.


An easy short term solution is to increase the width of the pen :

![larger]({{site.baseurl}}/assets/img/saxi_larger_pen.png)



#### Preparing for a larger frame

In order to an interesting final results, the drawing must be sufficently large. However the training time for a single image of this size is long, about 30 minutes with a high end gpu.

In order to facilitate this optimization process, one trick was to create a starting image good enough to ease process.
- Optimize for a size 256x256
- Replicate the given image to fill a 1024x1024 
- Finetune the large image

![pretraining]({{site.baseurl}}/assets/img/pretraining_small.png)

Before the pretraining starts


![pretraining]({{site.baseurl}}/assets/img/after_finetuning.png)

After the finetuning


## Real life result

![final render]({{site.baseurl}}/assets/img/final_render_neural_style_axi.jpg)

The final result is big enough to see the pattern multiple times but the eye cannot catch the small irregularities too easily


![in a frame]({{site.baseurl}}/assets/img/render_in_a_frame.jpg)

In a frame, the result looks kind of professional.


![another example]({{site.baseurl}}/assets/img/another_example.jpg)

Another interesting pattern.


![worn up]({{site.baseurl}}/assets/img/worn_up_pen.jpg)

Here the pen got tired during the \~1h30 of drawing. 


![highlighter proto]({{site.baseurl}}/assets/img/early_prototype_with_highlighter.jpg)

Result on a smaller piece of paper with an highlighter, the paper quality is important in order to absorb all the ink coming for the repetitive stroke in the same area.
