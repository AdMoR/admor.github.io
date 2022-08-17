# Post hoc color quantization for pen plotting


## Introduction

In the previous post, we have seen ideas to handle plottable colors in neuron excitation.

However, we remained at a very basic level of transformation. The quantization process can be very rough and does not respect the transparency that was used in the original optimization process.

In this blog post, we will deep dive on methods for truthful post hoc color quantization process.

In short : 
1 - We improve the basic algorithm for quantization and observe the qualitative results
2 - Using an alternative generation method, we improe the previous method with transparency handling


## TLDR

We get this kind of production

![]()

![]()





## Stroke based images and its shortcomings


#### Current state of affair

Optimization using free form curves 

......

![shortcoming in simplification]({{site.baseurl}}/assets/img/loss_of_magic_after_quantization.png){: width="500" }


Given the unsatisfactory results on the previous color quantization methodology, we want to rethink our approach.

In the first part, we avoided one main element : 
- the color interaction between curves through the alpha channel

This made our problem much easier because : 
- computing multiple intersections and covers is hard
- the curves are 3rd order bezier, making their intersection not regular polygons

Despite the difficulty, we pursue this direction as the main improvment possibility. 



#### Using polygon as optimization basis

From the previous section, we understood that we need to be able to compute the precise intersection of shapes in order to assign the right quantized color.

One possible solution is to change the optimization basis from curves to polygon. And this is totally doable with Diffvg.

From there, we need to rework the svg generation process to use this different basis.

![generation using polygons]({{site.baseurl}}/assets/img/polygon_neuron_excitation.png){: width="500" }
![generation using polygons]({{site.baseurl}}/assets/img/polygon_neuron_excitation_2.png){: width="500" }



## Color quantization

Before diving in the algorithm, let's sum up the problem.

```
Input :  We have N polygon with different colors and stroke size
Output : We have the same set of polygon but with colors and stroke limited to a given collection C_n
```

This is needed as we can use only a limited number of pens.

#### K-means

The different colors used on the curves are 4 dimensional : 3 RGB channel plus an alpha one.

The alpha one is important in our problem as it allows to have several curves on the same spot contributing to a color mix.
However as most of the space is filled with curve that are non intersecting, we can transform the 4-channels into only 3 by multipling with the alpha.


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

![show color palette]({{site.baseurl}}/assets/img/){: width="750" }

One detail on the color, using [Lab]() is crucial to have clusters that are natural to the human eye.
![with and without lab]()

That's it for the basics.


#### Ransac

RANSAC stands for .... It is a technic used to remove outliers.

Why do we need it ?

Because a few curves happen to have colors completely off the limited palette we would like to have.

K mean may not be resistant to this kind of data as its centers may shift in the direction of the outlier or even occupy a center if enough colours are allowed.
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


![Palette difference]({{site.baseurl}}/assets/img/color_palette_difference_with_ransac.png){: width="750" }

![Color change  after ransac]({{site.baseurl}}/assets/img/difference_in_rendering_with_ransac.png){: width="750" }



## Producing the final print

So far we used polygons to represent the shapes in the image. However, this is not enough when we want to plot it with pen.

When viewing an image, layering is taken into account, plotter tools don't take this into account, this must be computed manually. 

In short, we need to carve away for each layer the space occupied by the layer above.

![layer cut example]({{site.baseurl}}/assets/img/layer_cuts.png){: width="750" }

In this example, we have 3 full square of different color in the first image. The 3 subsequent images show how we should transform the squares to be able to plot the image.




