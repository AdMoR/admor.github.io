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


![im3](/assets/images/voronoi_cells.png)



### The code 


Do it and see by yourself that it is not a very powerful way to do it.


```python
from matplotlib import pyplot as plt
%matplotlib inline
import time
import numpy as np
from sklearn import cluster
from skimage import io
from skimage.color import rgb2lab, lab2rgb
from skimage import data


def cluster_colors(rgb_img, n_clusters=10, color="rgb"):
    km = cluster.KMeans(n_clusters=n_clusters)
    img = rgb_img
    km.fit(img.reshape(-1, 3))
    
    return km

def build_spatial_clustering(img, xs, ys):
    pos_km = cluster.KMeans(n_clusters=len(xs))
    pos_km.cluster_centers_ = np.concatenate([ys.reshape(-1, 1), xs.reshape(-1, 1)], axis=1)
    all_points = [np.array([y, x]) for x in range(img.shape[1]) for y in range(img.shape[0])]
    cluster_attachment = pos_km.predict(all_points)
    
    return all_points, cluster_attachment

def get_cluster_mean_colors(img, all_points, cluster_attachment, n_points):
    mean_colors = list()
    for i in range(n_points):
        mean_colors.append(list())

    for pos, cluster_idx in zip(all_points, cluster_attachment):
        mean_colors[cluster_idx].append(img[pos[0], pos[1], :].reshape(1, -1))
        
    return mean_colors

def build_simplified_img(img, all_points, cluster_attachment, color_per_zone, km):
    clustered_img = np.zeros(img.shape)
    
    print(max(cluster_attachment))
    print(len(color_per_zone))
    
    all_colors = np.array([color_per_zone[cluster_idx] for cluster_idx in cluster_attachment])
    color_indexes = km.predict(all_colors)
    
    for idx, (pos, cluster_idx) in enumerate(zip(all_points, cluster_attachment)):
        clustered_img[pos[0], pos[1], :] = km.cluster_centers_[color_indexes[idx]]

    clustered_img = np.round(clustered_img / 255., 2)
    return clustered_img



def simplify_img(img, n_points, n_colors):
    # Image shape
    tt = time.time()
    h, w, c = img.shape
    print(img.shape)
    
    # Find main color of the image 
    km = cluster_colors(img, n_colors)
    
    # Create the local sampling
    xs = np.random.randint(w, size=n_points)
    ys = np.random.randint(h, size=n_points)
    all_points, cluster_attachment = build_spatial_clustering(img, xs, ys)
    
    # Get mean color per cluster
    mean_colors = get_cluster_mean_colors(img, all_points, cluster_attachment, n_points)
    
    # Get the mean color in each spatial zone
    color_per_zone = list()
    for color_array in mean_colors:
        mean_color = np.mean(np.concatenate(color_array, axis=0), axis=0) if len(color_array) > 0 else np.array([0, 0, 0])
        color_per_zone.append(mean_color)
    
    print(time.time() - tt)
    # Replace mean color per zone by the simplified color
    final = build_simplified_img(img, all_points, cluster_attachment, color_per_zone, km)
    print(time.time() - tt)
    return final 
    
def show_result(img):
    clustered_img = simplify_img(img, n_points, n_colors)

    plt.figure()
    plt.imshow(clustered_img, interpolation='nearest')

    plt.figure()
    plt.imshow(img, interpolation='nearest')

    plt.show()



url = "https://s1.qwant.com/thumbr/0x380/2/b/825b7cfdd7fe089e8bf40214f2a23820aa136a0f77a6497cfbd132649c16ba/fo2m9ipgvt901.jpg?u=https%3A%2F%2Fi.redd.it%2Ffo2m9ipgvt901.jpg&q=0&b=1&p=0&a=1"
img = io.imread(url)
n_points = 1000
n_colors = 20
show_result(img)
```


