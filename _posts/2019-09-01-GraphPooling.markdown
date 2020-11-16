---
description: How can CNN architecture be applied to graph structures ?
tags: convolutional neural networks graph pooling 3D
---

## Introduction on graph convolutional neural nets

Convolutional neural nets have been very successful for image, text and sound. You probably heard about them for graph too, so how do they work ? That's what we gonna see.

But first let's see some applications.

#### Graph auto encoder

#### ....



## Graph convolution

Convolution is well defined when there is a notion of space. But for graph, unfortunatelly,  there isn't one.
So 


## Graph pooling

The notion of pooling in graphs is more challenging. Again a notion of proximity needs to be defined.
The Graclus algorithm uses the weights on the edges to define close vertices and merge them.

#### Merge steps




#### Tree reconstruction

Once we managed to build the last graph layer, eg when only one vertex is remaining. 
In this operation, we need to transform our hierarchy into a balanced binary tree, this process will create fake nodes that will allow to have a regular structure for the pooling operation on the original vertices.

![]()

Thanks to this structure, we can use a normal 1-D pooling operation from neural net libraries.
In this example, we create ????? new nodes .....


#### How to adapt for graph variation

As for fully convolutional neural nets, after some pooling operations, a global pooling step could be used.
But as for traditionnal CNN, performances will be higher if all graphs share approximaly the same structure.



## Nice articles on graph convolution

##### Graph Convolutional Neural Networks for Web-Scale Recommender Systems

Main steps : 
- Localized convolution : Use a sub part of the graph for conv
- Convolution with random walk : sampling of the neighbourhood of nodes using a random walk importance sampled with edge weights
- Importance pooling : Use random walk to generate weights that can be used to do sample pooling
- Curriculum training and efficient map reduce schema


Neighbourhood feature is defined as the weigthed mean of node feature thanks to the random walk extracted weights.

Pooling done by inversion : for batch b, we select n_batch nodes N and wish to stack K convolutions. 
We thus proceed to build S(k) recursively with S(K) = {N} and S(k-1) = S(k) + neighbourhood(k)

for each phase k, neighbourhood convolution is then performed on neighbourhood of each node of S(k).
After it, for each node, the embedding of the next layer is computed with the current the current embedding of the node and the one of the neighboorhood.

The article provides a clean summary of the algorithm.
![Algorithm_2](Algorithm_2.png)