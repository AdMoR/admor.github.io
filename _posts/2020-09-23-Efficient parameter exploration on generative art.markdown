---
description: and how my workflow evolved
tags: generative algorithm pattern visuallization
---


I've played for some time with different kind of generative algorithm to produce more or less successfully beautiful image.

[This post](https://admor.github.io/2020/01/10/Photo-to-drawing-Part-1.html) can attest it. It can be difficult to get a grasp of how good an idea can be without trying a lot of different combinations.


## Evolution of the design

### Presentation of the algorithm

When i started ancient scribble, I wanted to play with dichotomy and tree structure and see what I could do by myself and without external ideas.

General idea : 
- Create recursively areas using dichotomy from the original canvas
- For each created area, sample N points based on the area of the rectangle
- Link the points using a greedy algorithms, skip the points to far away
- Smooth the junctions of points using chaikin curves


### First tools to test configurations 

I started to add sliders to control parameters of the generation.

![captions](/assets/images/ancient_scribbles.png)

It worked correctly to test a variety of settings but this is not perfect.
- Changing parameters one by one is slow
- You need to define a good range of value for your slider


### New features and the overflow of parameters

As i started to grow unsatisfied of the dull looking patterns, i tried to add more and more parameters to find a flavour valuable to my eye.

![too many](/assets/images/too_many_params.png)

But after reusing the codebase after a few weeks it started to look


### Using the mouse

After reading some examples from the [Generative Design website](http://www.generative-gestaltung.de/2/) i realised that using the mouse instead of a slider made a lot of sense.
Despite the fact that you can represent only two slider with your mouse movement, these sliders are very precise and intuitive.

I removed all the sliders and explored which parameters were the most important for muy generation.
I ended up creating a new parameter which provided much more interest : point density.
Why ? While playing with the mouse, I could easily see that some parameters would bring very little variation between generation.

![Change across the spectrum](/assets/images/scribbles-crop.gif)
  



## Miscellaneous

### An algorithm for cutting a quadrilateral

TODO

### Printing with AxiDraw

TODO

