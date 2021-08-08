---
description: Generate crazy images
tags: python image processing CLIP VQGAN
img: robot_love.png
comments: true
---

## Introduction

Very often, AI promises don't deliver. But sometimes they deliver much beyond your expectations. CLIP VQGAN is one of them.


## What can you create ?

#### Some examples

![I'm in love with a robot](/assets/img/robot_love.png)
I'm in love with a robot

![Back to the pit](/assets/img/back_to_the_pit.png)
Back to the pit

![Chicken_surf](/assets/img/chicken_surf.png)
Chicken surf

![Toaster_forest](/assets/img/toaster_forest.png)
Toaster forest

![the_rave](/assets/img/the_rave.png)
The rave

![colorful_mushrooms](/assets/img/colorful_mushrooms.png)
Colorful mushrooms

![chicken_boulder_land](/assets/img/chicken_boulder_land.png)
Chicken boulder land



#### Where can you do it yourself ?

There is an easy access to [a colab notebook](https://colab.research.google.com/github/dribnet/clipit/blob/master/demos/Start_Here.ipynb#scrollTo=XziodsCqVC2A).

I had a lot of fun with it.



## What are the details of the algorithm ?

#### CLIP 

CLIP has been developped by OpenAI as a way to bridge the gap between image and textual representation at a large scale. This allow them to learn a very performant zero-shot visual model (they don't need to label any image manually).

As it comes from OpenAI there is already a lot of litterature covering the subject, like [their blog post](https://openai.com/blog/clip/)

What I was curious about was the implementation used in order to get the powerful embedding they are getting.
The following screenshot from their paper describes how easy the general idea is.


![clip_algo](/assets/img/clip_algo.png)


They use : 
- A very large dataset of image and caption couples
- A slightly modified resnet for image features and a transformer for text embedding
- An embedding transformation is learned for each source of information
- A softmax among the batch forces the network to learn what is the most appropriate sentence for each image


#### VQGAN

This is a mix between a transformer and a GAN architecture.

![An example](/assets/img/example_VQGAN.png)

Some useful links : 
- [A very simple summary of the paper](https://t.me/casual_gan/46)
- [The github of the paper](https://github.com/CompVis/taming-transformers)

In short : 
- The work of the generator is made simpler with a codebook of quantized features
- A transformer learns to associates which codes go well together
- Codes from the geenrated image are constructed in a sliding window manner
- The generator only has to learn to reconstruct a code to a real image


#### Generation 

**Forward** : 

The generated image goes through the image path of the CLIP model and the embedding is compared to the text embeding. 
We maximise the similarity between the two.
To have a better looking image, a batch of crop and different data augmented version of the image are passed in a batch.

**Backward** : 

So what do we optimize exactly ?

In fact, the optimized image is parametrized by the VQGAN internal representation. 
We start with random noise and pass it through the Vqgan model.
The total backward loss is computed through all these networks.



## Appendix

#### Nightmare material

What everyone didn't need !

![nightmare](/assets/img/no_one_reaches_the_end.png)
![nightmare](/assets/img/mocking_face_in_the_forest.png)
![nightmare](/assets/img/toad_wizard_and_the_witch.png)
![nightmare](/assets/img/possessed_meat_cupcake.png)

#### Faery tale characters

What does a paper company cat ceo looks like ?

![faery](/assets/img/dog_engineer_from_the_love_department.png)
![faery](/assets/img/toad_witch.png)
![faery](/assets/img/the_paper_company_cat_ceo.png)
![faery](/assets/img/peacock_board_of_directors.png)
