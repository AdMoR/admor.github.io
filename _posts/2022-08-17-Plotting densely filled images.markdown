---
description: The path to plot with richer content
tags: python svg neuron excitation optimization pytorch
img: poly_gen_result_1.JPG
comments: true
---


## Introduction

In the previous post, we have seen ideas to handle plottable colors in neuron excitation.

However, we remained at a very basic level of transformation. 
The quantization process can be very rough and does not respect the transparency that was used in the original optimization process.

In this blog post, we will dive deeper on methods for truthful post hoc color quantization process.

In short : 

1 - We improve the basic algorithm for quantization and observe the qualitative results

2 - Using an alternative generation process, we improve the previous method with transparency handling


## TLDR

We get this kind of production

![result_1]({{site.baseurl}}/assets/img/poly_gen_result_1.JPG){: width="750" }

![result_2]()



## Stroke based images and their shortcomings

#### Review of the plots from the previous posts

Optimization using free form curves gives initially interesting pictures.
But during the postprocessing that allows to make it plottable, we lose the magic touch that makes it interesting.


![shortcoming in simplification]({{site.baseurl}}/assets/img/loss_of_magic_after_quantization.png){: width="750" }

In this example, we quantize the number of color and stroke width.


#### What's next ?

Given the unsatisfactory results on the previous color quantization methodology, we want to rethink our approach.

We avoided two main elements in the previous approach : 
- curves are more than points, they are in fact polygons
- the color quantization should take into account the mixing of color between curves

The problem remains hard with this new setup : 
- computing multiple intersections and covers is hard
- the curves are 3rd order bezier, making their intersection not regular polygons

Despite the difficulty, we pursue this direction as the main improvment possibility.

#### Using polygon as optimization basis

From the previous section, we understood that we need to be able to compute the precise intersection of shapes in order to assign the right quantized color.

One possible solution is to change the optimization basis from curves to polygon. And this is totally doable with Diffvg.

From there, we need to rework the svg generation process to use this different basis.

![generation using polygons]({{site.baseurl}}/assets/img/polygon_neuron_excitation.png){: width="400" }
![generation using polygons]({{site.baseurl}}/assets/img/polygon_neuron_excitation_2.png){: width="400" }

#### Reducing the number of used polygons with LIVE

[LIVE](https://github.com/Picsart-AI-Research/LIVE-Layerwise-Image-Vectorization) is an improvement over diffvg for painterly rendering.

Using a few tricks, the same image can be produced using far less polygons : 

- Incremental addition of new shapes along a partial optimization process
- Shape positions are not initialised at random rather depending on where the reconstruction error is maximum
- A new constraint on curves to avoid self interaction 

This will help a lot as the intersection problem is at least O(n2)

![example of live productions]({{site.baseurl}}/assets/img/live_method_productions.png){: width="800" }


## Color quantization

This is one of the main step.

Because finding the right methodology is required in order to produce a simple yet truthful plot of the original image, we present some ideas on the quantization process.

```
# Outlook of the algorithm
Input :  We have N polygon with different colors and stroke size
Output : We have the same set of polygon but with colors limited to a given collection C_n
```


#### K-means

The different colors used on the curves are 4 dimensional : 3 RGB channel plus an alpha one.

The alpha one is important in our problem as it allows to have several curves on the same spot contributing to a color mix.

The final algorithm is then pretty straight forward :
```
# Get color from each curve, in Lab format and alpha removed
X = [rgb2lab(c.color[:3] * c.color[:3]) for c in curves] 
# Stroke width weighting allows to have larger stroke color better represented
W = [c.stroke_width for c in curves]
kmeans = KMeans(n_clusters=n_quantized_colors)
kmeans.fit(X, sample_weights=W)
display_colors(kmeans._centers)
```

That's it for the basics.


#### Ransac

RANSAC stands for .... It is a technic used to remove outliers.

Why do we need it ?

Because a few curves happen to have colors completely off the limited palette we would like to have.

K-means may not be resistant to this kind of data as its centers may shift in the direction of the outlier or even occupy a center if enough colours are allowed.
However outlier removal should be used carefully as sparsely used color can sometimes be fundamental in the rendering of the image.

![example of image with outlier colours]()

What is the pseudo code for our ransac ?

```python
k = ... #n umber of sampled colors
X, W = ... # curve colors and curve strokes used as weighting
N = X.shape[0] # The number of curves
for i in ransac_tries:
    n_sampled_this_round = randint(0.5 * N, N)
    index_sampled = random.choice(range(N), n_sampled_this_round, replacement=False, p=W)
    X_sampled, W_sampled = X[index_sampled], W[index_sampled]
    kmeans = Kmeans(n_clusters=k).fit(X_sampled, sample_weight=W_sampled)
    score = kmeans.score(X_sampled, W_sampled) / sum(W_sampled)
    results.append((kmeans, score))
return sorted(results, key=lambda x: x[1], ascending=False)[0]
```

The main steps :
- Sample n out of N curves based on the weights as density
- Fit a new kmeans with this n elements
- Compute the average score (= agreement among the sampled points on the quantized colors)
- Finally keep the kmeans that has the best score


![Palette difference]({{site.baseurl}}/assets/img/color_palette_difference_with_ransac.png){: width="450" }

![Color change  after ransac]({{site.baseurl}}/assets/img/difference_in_rendering_with_ransac.png){: width="800" }

We often get cases where a color is replaced by another and it improves the final results.


## Computing regions covers 

So far, we have seen how color quantization can be done given our set of shapes.

But we did not handle very seriously the transparency.

#### Pixel cover

One way to handle transparency is to use the pixelated rendering of svg.

The uniform grid sampling guarantees that the collected colors will be representative of the final rendering.

From the previous code snippet, we need to change only two things :

```
X = rendered_image.reshape(-1, 3)
W = None
```

![pixel cover]({{site.baseurl}}/assets/img/the_importance_of_transparency_in_color_computation.png){: width="400"}
![origin image]({{site.baseurl}}/assets/img/color_sampled_from_this.png){: width="400"}

The change is simple and produce meaningful color when looking at the original image.

**So why didn't we do it IN THE FIRST PLACE ?**

Because it cannot be simply applied to our shapes. We need to backtrack what pixel values mean at the shape level.

Luckily, this is what we will do in the next section.


#### Shape cover

So far we used polygons to represent the shapes in the image. 
However, this is not enough when we want to plot it with pen.

When viewing an image, layering is taken into account, plotter tools don't take this into account, this must be computed manually. 

In short, we need to carve away for each layer the space occupied by the layer above.

![layer cut example]({{site.baseurl}}/assets/img/layer_cuts.png){: width="750" }

In this example, we have 3 full square of different color in the first image. The 3 subsequent images show how we should transform the squares to be able to plot the image.

From this basic idea, we compute the final image as :

- a set of intersections of all shapes
- the difference of all shapes from these polygons


**Algorithm**

![poly cut algorithm run]({{site.baseurl}}/assets/img/polygon_section_cuts.JPG){: width="750" }

Details : 

- We start with 3 shapes O1, O2, O3 : O1 being the higher up
- __treat__ function compute the required intersections
- first line depicts what happens when the function is run on 01
  - We get two intersections A and B
  - A and B must be removed from O1, O2 and O3 as they will hold different colors
  - A and B are pushed on top of the priority list 
- next, A is treated 
  - its intersection with B, named C, is an intersection with 3 shapes
  - Again C is removed from B and pushed to the priority list
  - No more intersection exist between C and another polygon, regular operation will resume
- B, D, O2 and O3 are regular or similar cases

