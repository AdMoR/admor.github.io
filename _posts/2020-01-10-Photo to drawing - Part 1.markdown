---
description: The fast and dumb way
tags: image processing stylisation python 
---

The goal : from an any photo, find a representation of the image that could be drawn.

Example :

![color book](https://s1.qwant.com/thumbr/0x0/2/6/3281961e818398a19221452d018af6b9da5bc0072a894a93e8c6d7a43cb1ea/black-white-fantasy-picture-sun-260nw-577386661.jpg?u=https%3A%2F%2Fimage.shutterstock.com%2Fimage-vector%2Fblack-white-fantasy-picture-sun-260nw-577386661.jpg&q=0&b=1&p=0&a=1)

From this, we could add colour and have a perfectly drawable representation of our original landscape.


# A fast implemntation

3 easy steps :
- Sample N random spatial clusters in the image
- For each cluster, find the mean color
- Reduce the number of colours by clustering the color of the image


### Results 


![im3](https://s1.qwant.com/thumbr/0x380/2/b/825b7cfdd7fe089e8bf40214f2a23820aa136a0f77a6497cfbd132649c16ba/fo2m9ipgvt901.jpg?u=https%3A%2F%2Fi.redd.it%2Ffo2m9ipgvt901.jpg&q=0&b=1&p=0&a=1)

![im3_tf](/assets/images/im3_transformed.png)
