---
description: Low hassle way to display your mesh data
tags: mesh data 3D tensorflow 
---

3D mesh is still underused but very promising as much closer to a real world model.
However the setup cost is high without the right setup.
My goal here is to offer a way to quickly check your mesh data with tensorflow.


### 1. Setup a tensorflow, jupyter and tensorflow graphics container

The basic are provided by a tensorflow docker. So it is very easy to iterate on top of it and add the graphics extension.

```
FROM tensorflow/tensorflow:latest-gpu-py3-jupyter
MAINTAINER Admor
RUN apt-get update && apt-get install -y git
RUN apt-get install -y libopenexr-dev && pip install --upgrade OpenEXR
RUN pip install tensorboard
RUN git clone https://github.com/tensorflow/graphics.git && cd graphics && sh build_pip_pkg.sh && pip install --upgrade dist/*.whl
```

This will build a container with exactly what we need.
It can be launched with something like 
`sudo nvidia-docker run --rm -it --init --runtime=nvidia --ipc=host  --volume=$PWD:/app -e NVIDIA_VISIBLE_DEVICES=0 -p 6006:6006 -p 8888:8888 bf50632bc15c`


### 2. Loading the mesh data

A few lines of python allow to easily load OFF format 3D mesh data.
You can try the following script.


```python
import matplotlib.pyplot as plt
import tensorflow as tf
from tensorflow_graphics.math.interpolation import bspline
from tensorflow_graphics.math.interpolation import slerp
tf.disable_eager_execution()
from tensorboard.plugins.mesh import summary as mesh_summary
import numpy as np

def read_off(file):
    if 'OFF' != file.readline().strip():
        raise('Not a valid OFF header')
    n_verts, n_faces, n_dontknow = tuple([int(s) for s in file.readline().strip().split(' ')])
    verts = [[float(s) for s in file.readline().strip().split(' ')] for i_vert in range(n_verts)]
    faces = [[int(s) for s in file.readline().strip().split(' ')][1:] for i_face in range(n_faces)]
    return verts, faces

get_name = lambda name: tf.get_default_graph().get_tensor_by_name(name + ":0")
vert, faces = read_off(open("/app/airplane_0001.off"))
format_ = lambda l: np.expand_dims(np.array(l), 0)
vert_tensor = format_(vert).astype(np.float32)
faces_tensor = format_(faces).astype(np.int32)


vertices_tensor = tf.placeholder(tf.float32, vert_tensor.shape, name="a")
faces_tensor = tf.placeholder(tf.int32, faces_tensor.shape, name="b")
log_dir ="./my_mesh"

meshes_summary = mesh_summary.op(
    'mesh_color_tensor', 
    vertices=vertices_tensor, 
    faces=faces_tensor,
)

# Create summary writer and session.
writer = tf.summary.FileWriter(log_dir)
sess = tf.Session()

summaries = sess.run([meshes_summary], 
    feed_dict={
    get_name("a"): [vert],
    get_name("b"): [faces],
})
# Save summaries.
for summary in summaries:
    writer.add_summary(summary)
```

I got thre follwoing result with a sample from the ModelNet dataset ()()

![mesh loaded](/assets/images/mesh_load.png)