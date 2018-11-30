---
description: I tried neuron excitation through direct optimization and it gives great visualizations !
tags: neuron excitation visualization deep learning
---

The basic idea behind feature visualisation is to find an image that maximize the value of a given neuron in a neural network.

### How does it work ? 

We initially train a neural network function to recognize concepts (usually ImageNet with 1000 object classes), let's say `AlexNet`.

![AlexNet architecture](https://res.mdpi.com/remotesensing/remotesensing-09-00848/article_deploy/html/images/remotesensing-09-00848-g001.png)

In this neural network, we want to know what activates a specific layer, let's say `Conv5`. We will try to find an image `I` that maximize the norm of this layer.
Our loss is going to be something like : 

$$ \mathcal{L}_{index}(I) = -  1/n_{features} || \mathcal{F}_{conv_5}(I)[:, index, :, :] ||^{2} + \lambda  regularization(I) $$

We use back-propagation to have an image that will gradually get a lower score.
The neural network is differentiable and thus we can get a clean gradient to update our image.

$$ I -= grad_I(\mathcal{L}_{index}(I)) $$

Depending on the network, this gives more or less interesting pics.


### What does it look like ?

This is where things get complicated. With basic optimization, we mostly get noise : 
![AlexNet Conv5 noise](/assets/images/alexnet_noise.jpg)


That's where our `regularization(I)` comes into play. We need it to remove the high frequency noise and keep only the image structure. To do so, we use total varialtion loss.

$$ regularization(I) = TV(I) = \Sigma_i ||I(i+1) - I(i)||^2 $$

With the right `lambda`, we can still have the structure and remove most of the noise.
It looks different from network to network and layer to layer, but we can get satisfying results.
Let's see some examples


AlexNet, different layers


![AlexNet Conv5 layer 100](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/LayerExcitationLoss_alexnet_1_15_2048_0.0005.jpg)
![AlexNet FC3 neuron 100](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/LayerExcitationLoss_alexnet_1_18_2048_0.0005.jpg)
![AlexNet ](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/LayerExcitationLoss_alexnet_1_34_2048_0.0005.jpg)

Last layer, VGG16


![VGG16 FC3 neuron 100](https://raw.githubusercontent.com/AdMoR/neural-styles/master/images/LayerExcitationLoss_vgg16_-1_4_2048_0.1_0.0005.jpg)


As we can see, the networks generate totally different images. But we can recognize some things on last layer (close to the object prediction of ImageNet like frog).


