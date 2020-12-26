---
description: For things like qunatization of colors and many other
tags: python numpy image
img: quantization.png
comments: true
---

I often look for the same answer for the same questions on the internet.
This is my way to remember them more easily. 


### Fill a matrix based on an index map

Typical usage would be : I need to update an image based on a computation that has the same size than my image.
Doing a for loop is prohibited. What is the matricial way of doing it ?


```python
# Simulation of the output of an algorithm
# it has labels for all positions
label_map = np.random.randint(0, 2 * 10, (10, 10))

# A black color image
img = np.zeros((10, 10, 3))

# Business logic : pixel whose label has been assigned to 1 should be white
xs, ys = (label_map == 1).nonzero()
img[xs, ys] = (1, 1, 1)
```


### Quantize an image colors

To reduce the complexity of an image when applying other algorithms.
Somre advanced methods for quantization are discussed in this [stack overflow thread](https://stackoverflow.com/questions/49710006/fast-color-quantization-in-opencv)

```python
from PIL import Image  
import PIL  
import numpy as np
from matplotlib import pyplot as plt

    
im1 = Image.open("my_img.png").convert("RGB") 
im2 = im1.quantize(16)  

pix = np.array(im2.convert("RGB"))
print(pix.shape)
plt.imshow(pix)
```

An example of this processing can be found with this image 

![Quantization example](/assets/img/quantization.png)



### Unpacking a list of tuple to a tuple of lists

Pretty common but much more efficient when not using the for loop.

```python
l = [(1, 2), (4, 4)]
x, y = zip(*l)
x
#>>> (1, 4)
```

### Storing and using static files in your python tests

There is a simple way to get your test data with relative paths. 

```python
THIS_DIR = os.path.dirname(os.path.abspath(__file__))
test_data = os.path.join(THIS_DIR, "fake_data.csv")
```

