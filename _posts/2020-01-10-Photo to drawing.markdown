---
description: The fast and dumb way
tags: image processing stylisation python 
img: voronoi_cells.png
---

The goal : from an any photo, find a representation of the image that could be drawn.

Example :

![color book](https://s1.qwant.com/thumbr/0x0/2/6/3281961e818398a19221452d018af6b9da5bc0072a894a93e8c6d7a43cb1ea/black-white-fantasy-picture-sun-260nw-577386661.jpg?u=https%3A%2F%2Fimage.shutterstock.com%2Fimage-vector%2Fblack-white-fantasy-picture-sun-260nw-577386661.jpg&q=0&b=1&p=0&a=1)

From this, we could add colour and have a perfectly drawable representation of our original landscape.


# A fast implemntation

3 easy steps :
- Sample N random spatial clusters in the image
- For each cluster, find the mean color
- Reduce the number of colours to M by clustering the color of the image


<br/>
<br/>

### Results 


![im3](https://s1.qwant.com/thumbr/0x380/2/b/825b7cfdd7fe089e8bf40214f2a23820aa136a0f77a6497cfbd132649c16ba/fo2m9ipgvt901.jpg?u=https%3A%2F%2Fi.redd.it%2Ffo2m9ipgvt901.jpg&q=0&b=1&p=0&a=1)

<br/>
The result with 1000 clusters and 20 colors
![im3_tf](/assets/images/im3_transformed.png)


<br/>
<br/>

# Bonus : drawing only the boundary 

If the region are small enough, drawing the boundary of the cell could be enough to get the idea.  
To do so, we can follow these 4 easy steps :
- Get the eqaation of the mediator line for all pair of centers
- For each point of the line, determine if their are indeed closer to the two centers than the others. These point are the boundaries we are looking for.
- Plot two line for each line, one of each center color.


![im3]({{site.baseurl}}/assets/images/voronoi_cells.png)


<br/>
<br/>

### The code 

Do it and see by yourself that it is not a very powerful way to do it.  
However the performance of the raw implemntation is pretty poor. It takes large amount of time for small images and the scaling is horrible.  
Part 2 of this post will look at solving these issues.

The code is available [here](https://github.com/AdMoR/PlotterExperiments/blob/master/notebooks/voronoi%20and%20co.ipynb)