---
description: Going further on image generation with conditioning
tags: neural network feature visualization image processing
---

## How to regularize the generated image

In the previous article, we have seen how to generate images that maximize an intermediate layer of a neural network.

If you remember the optimization formula, you noticed the total variation term $$ TV(I) = \Sigma_i ||I(i+1) - I(i)||^2 $$.
This term is relatively empirical, we know it will reduce high frequencies by construction, but we don't control its effect on the image. Moreover its effect on RGB images may not be as we expect (because of the non linear perception of our eye).

I'm going to quickly review what was presented in this two articles [https://distill.pub/2017/feature-visualization/][1] [https://distill.pub/2018/differentiable-parameterizations/][2] and explain how we can improve our regularization.

#### Image transformations as regularization
The explored solution will be described with some pytorch code.

One way to have a stable image that maximize a feature layer is to force the image to have almost invariant activation across few pixel translation and few degree rotation.

To be able to do that, we need an original image that will be optimized and a set of transformations that will be applied to this original to create a batch of the same modified image. Exactly like when training a neural net for classification, we can use the data augmentation trick to better learn an image representation.

![one to many transformation](https://proxy.duckduckgo.com/iu/?u=https%3A%2F%2Fcdn-images-1.medium.com%2Fmax%2F1000%2F1*C8hNiOqur4OJyEZmC7OnzQ.png&f=1)

Let's have a look at how we can do the translation : 

```python
def jitter(tau, img):
    B, C, H, W = img.shape
    tau_x = random.randint(tau, 2 * tau)
    tau_y = random.randint(tau, 2 * tau)
    padded = torch.nn.ReflectionPad2d(tau + 1)(img)
    return padded[:, :, tau_x:tau_x + H, tau_y: tau_y + W]
```

We use the original image, pad it and crop the same size with a small bias tau in x or y directions.
In fact, the intermediate tensor is built that can later backpropagate to the individual pixels of the base image. 

The code for the rotation and small scaling follows the same idea.

```python
def scaled_rotation(x, in_theta=None, scale=None):

    if in_theta is None:
        in_theta = random.choice(list(range(-8, 9)))
    rad_theta = math.pi / 360 * in_theta

    if scale is None:
        scale = random.choice([0.95, 0.975, 1, 1.025, 1.05])

    if x.shape == 4:
        B = x.shape[0]
    else:
        B = 1

    theta = torch.zeros((B, 2, 3))
    theta[0, 0, 0] = math.cos(rad_theta)
    theta[0, 0, 1] = -math.sin(rad_theta)
    theta[0, 1, 0] = math.sin(rad_theta)
    theta[0, 1, 1] = math.cos(rad_theta)

    grid = scale * F.affine_grid(theta, x.shape)
    return F.grid_sample(x, grid)
```

The useful tool here is the grid_sample function that allows subsampling with float factors and keeps differiability propagation.

```python
class Rotation(nn.Module):
    def __init__(self, in_theta=None, scale=None):
        super().__init__()
        B = 4
        theta = torch.zeros((B, 2, 3))
        for i in range(B):
            if in_theta is None:
                in_theta = random.choice(list(range(-8, 9)))
            rad_theta = math.pi / 360 * in_theta
            if scale is None:
                scale = random.choice([0.95, 0.975, 1, 1.025, 1.05])
            theta[i, 0, 0] = math.cos(rad_theta)
            theta[i, 0, 1] = -math.sin(rad_theta)
            theta[i, 1, 0] = math.sin(rad_theta)
            theta[i, 1, 1] = math.cos(rad_theta)
        self.grid = nn.Parameter(scale * F.affine_grid(theta, [B, 3, 224, 224]))
    def forward(self, x):
        y = torch.cat([x for _ in range(4)], dim=0)
        z = F.grid_sample(y, self.grid)
        return z

r = Rotation()
with SummaryWriter(log_dir="./logs") as w:
     w.add_graph(r, x)

```

![Graph of the scaled rotation](/assets/images/scaled_rotation_op.png)


This gives us the following series of operations. Our input image is given by the index 0, it is replicated in 4 on operation 2 and 4 different rotations and scale are applied on operation 5.


#### Original image preconditioning
Transformations are not the only way to regularize our image, we can also change the space from which it is optimized. Fourier space makes sense as a single change in the frequency domain impacts the whole image.

What is suggested in the [Distill article]() and present in the [Lucid Library]() is to optimize a complex Fourier image rather than the raw RGB space image. On top of that, the color channel should not just be RGB but taken from a decorrelated space (aka coming from a PCA like transformation). The decorrelated space to RGB is learned through PCA from ImageNet data, let's have a look at how we can do this.

```python
import torchvision
import numpy as np


d = torchvision.datasets.CIFAR10("./", download=True)
dd = d.train_data - np.mean(np.mean(np.mean(d.train_data, axis=0), axis=0), axis=0)

s = list()
for b in range(dd.shape[0]):
    for i in range(dd.shape[1]):
        for j in range(dd.shape[2]):
            v = dd[b, i, j, :].reshape(1, -1)
            s.append(v.transpose().dot(v))

final = np.stack(...)
```



