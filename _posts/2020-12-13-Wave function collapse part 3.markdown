---
description: Generation inspired by an example
tags: python generative algorithm numpy 
img: gen_synthesis.png
comments: true
---

This post is the thrid one on wave function collapse.

Note : Have a look at the original [github repository](https://github.com/mxgmn/WaveFunctionCollapse) implementing this algorithm.


# Going further than the simple tile model

In this posty, we are going to discuss the overlapping wave function collapse. 
The strength of this algorithm ? 
**No need to define the adjacency rules between the tiles !**

It means that one can easily pick an example and start generating !
In fact the most impressive generation are using this version of the algorithm.


## How do we adapt the algorithm ?

To me, at first, it was really unclear how the initial algotrithm could adapt to **overlapping** tiles.
The idea became clear when I made the analogy with convolution filters that have both a size and a **stride** (which is usually half of the size).

![Size and stride](https://i.stack.imgur.com/P0r8c.png) 

This image was taken from a website illustrating convolution neural networks but it is actually perfect for WFC.
The first image is the original image and the second image is the class map.
By reading the input image, we can learn that class RED is a possible left neigbour of class GREEN.

## Progamatically 

We need to do 2 things : 
- Compute the slices according to the stride value
- Compute connexion according to a stride-based overlap

After that, we get an adjacency matrix exactly like previously.
So the algorithm can run without any change to the logic.

Let's have a look of how the tile extraction works :

```python
class OverlappingTileSetBuilder:
    
    MAX_INDEX = 10

    @classmethod
    def compute_connexion(cls, img1, img2, stride, dir_i, dir_j):
        """
        Overlapping connexion implies that we are to match a large area.
        Example : size = 7, stride = 3

        img 1               img2

        xxxoooo           ooooooo
        xxxoooo           ooxoooo
        xxoooxx           ooooooo

        img1 --right-neighbour--> img2 ?
        
        Then :

        img1[:, stride:]    img2[: :-stride]

            ooo           ooo
            ooo           oox
            oxx           ooo

        connexion --> img1[:, stride:] == img2[: :-stride]
        """
        if dir_i == 0:
            if dir_j == 1:
                return (img1[:, dir_j  * stride:, :] == img2[:, :(-(dir_j) * stride), :]).all()
            else:
                return cls.compute_connexion(img2, img1, stride, -dir_i, -dir_j)
        else:
            if dir_i == 1:
                return (img1[dir_i  * stride:, :, :] == img2[:(-(dir_i) * stride), :, :]).all()
            else:
                return cls.compute_connexion(img2, img1, stride, -dir_i, -dir_j)

    @classmethod
    def build_tile_adjacency_from_tileset(cls, tileset, stride):

        max_index = len(tileset)

        adj = dict()
        for d in {(-1, 0), (1, 0), (0, -1), (0, 1)}:
            adj[d] = np.zeros((max_index, max_index))
            for i in range(max_index):
                for j in range(max_index):
                    adj[d][i][j] = cls.compute_connexion(tileset[i].tile, tileset[j].tile, stride, d[0], d[1])

        return adj

    @classmethod
    def build_slicer(cls, my_img, size, stride):
        """
        The number of tiles depends on stride
        """
        # We want to limit the size of the image where we collect patches
        my_img = my_img[:size * cls.MAX_INDEX, :size * cls.MAX_INDEX, :]
        max_i_index = my_img.shape[0] / stride
        max_j_index = my_img.shape[1] / stride

        def slicer(i, j):
            assert (i >= 0 and i < max_i_index)
            assert (j >= 0 and i < max_j_index)
            return my_img[i * stride: i * stride + size , j * stride: j * stride + size , :]
        return slicer
    
    @classmethod
    def build_adj_and_tileset_from_img(cls, img, size, stride):
        my_slicer = cls.build_slicer(img, size, stride)
        tile_set = list(map(lambda t: UniqueTile.from_tile(my_slicer(t[0], t[1])), 
                            [(i, j) for i in range(cls.MAX_INDEX) for j in range(cls.MAX_INDEX)]))
        my_reduced_tileset = UniqueTile.reduce_tile_set(tile_set)
        my_reindexed_tileset = [UniqueTile(i, t[1], t[2]) for i, (k, t) in enumerate(my_reduced_tileset)]
        adj = cls.build_tile_adjacency_from_tileset(my_reindexed_tileset, stride)
        return adj, my_reindexed_tileset     

```

However the final map computed by the WFC algorithm has to be rendered a bit differently to handle the stride.


# Some generation

The 4 quadrant generations show the original image, the classes repartition, the class map and the final generation.

![Pully example](/assets/img/gen_6.png)

![Octal example](/assets/img/octal_2.png)

![Urban example](/assets/img/urban_overlap_large_2.png)

![Mondrian test](/assets/img/gen_mon_0.png)

![Pixel art example](/assets/img/pixel_art_example_gen.png)

