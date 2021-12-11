---
description: A generic way to reduce the number of colors for image generation
tags: python neural style machine learning
img: color_quantization_excitation.png
comments: true
---

## Introduction

I have been recently playing with the [diffvg lib](https://github.com/BachiLi/diffvg). It allows to optimize a set of lines and get an output that optimizes specific constraints like this example.

![example](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/result_n_paths400_im_size224_n_steps1500_layer_nameVGGLayers.Conv4_3_layer_index2.svg)

However in this case, the number of colors is not controlled. And thus cannot be plotted wit a regular set of pens.
A constraint very important if you want to draw this drawing with an [Axidraw](https://axidraw.com).


## A simpler problem : neural style with quantization

As the library is quite complex, I decided to start small by doing the regular neuron excitation work with a limited set of colors.
In order to do so, one could proceed as follow : 

- The optimized image is of size [H, W, N_COLORS]
- A second tensor stores the colors for each color class
- The highest value across the third channel of the image determines the color as a given pixel


However using an argmax doesn't make the problem differentiable, but we can of course use a softmax there.


```python
self.class_matrix = torch.nn.Parameter(torch.tensor(np.random.randn(H, W, n_colors), requires_grad=True))
self.class_colors = torch.nn.Parameter(torch.tensor(np.random.randn(n_colors, 3) + 0.5, requires_grad=True))
img = torch.clip(F.softmax(self.class_matrix, dim=2) @ torch.sigmoid(self.class_colors), 0, 1)
```


## Results

![quantized](/assets/img/color_quantization_excitation.png)

This first image shows the difference between the true image with the strict one color rule (left) and the relaxed version used for optimization (right).
Color on the right have different shades for a given hue.


![index9](/assets/img/index9.png)
An image as seen during the optimization

![index9_true](/assets/img/index9_true.png)
The true image with classes


## Some tricks

#### Controlling the colors with a sigmoid

`torch.sigmoid(self.class_colors)` allows to keep the color in bound of what it should.


#### A spiky softmax

`torch.clip(F.softmax(3 * self.class_matrix, dim=2)` allows to have less color cheating


#### A penalty for non spiky class_matrix

`loss += 0.01 * torch.norm(F.softmax(pal_im.class_matrix, dim=2) ** 0.5, 1)`

Here, we try to make arrays like `[0.2, 0.2, 0.6]` have a higher norm than `[1.0, 0, 0]`. So we take the sqaure root of the softmax output of color classes image.


## Ressource

[The code for the images](https://github.com/AdMoR/neural-styles/blob/master/quantized_neuron_excitation.py)




