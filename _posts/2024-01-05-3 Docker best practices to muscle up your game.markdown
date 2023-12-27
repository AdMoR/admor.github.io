---
description: Faster, lighter and more elegant builds
tags: docker best practices devops
img: docker_illu.png
comments: true
---

Not so long ago, I realised something disturbing. 

I knew many best practices in different domains : python, how to train a good model, how to save money on the cloud.

But for Docker, things were not so clear. That was a clear indicator that I should probably learn a lot more about it.

After a few discussions and reading on the topic, I collected a small set of resources that should help everyone feel more confident that they use Docker properly


## The essentials of Docker

This part is an intro for the best practices.  
Without it, the article could feel incomplete and frustrating if you did not spend enough time on Docker.

#### Docker in a nutshell

Docker is a platform for containerization that revolutionized software development and deployment. 
At its core, Docker enables the packaging of applications and their dependencies into lightweight, portable containers, which can run consistently across diverse computing environments. 

Containers encapsulate an application along with its runtime, libraries, and dependencies, ensuring seamless deployment on any system that supports Docker. 
This approach enhances consistency between development, testing, and production environments, mitigating the notorious "it works on my machine" issue. 

Docker simplifies application management, accelerates deployment, and optimizes resource utilization by sharing the host operating system's kernel. 

#### Why Docker

Docker started to be used around 2013 and became a key skill to have in the tech industry.
However, this does not tell a lot on the "why" it became popular.

Docker gained widespread adoption for several reasons and here are some key factors:

- Lighter than Virtual Machines:

    Virtualization technologies, such as VMware and VirtualBox, were commonly used. 
    VMs provide full virtualization of an entire operating system, allowing multiple applications with their dependencies to run on a single physical server. 
    While effective, VMs are heavier than containers in terms of resource usage, and the startup time is typically slower.

- Simpler than traditional configuration management tools:

    Tools like Puppet, Chef, and Ansible were used for automating the configuration of servers and ensuring consistency across different environments. 
    These tools focused on managing the software configuration on servers but didn't provide the same level of isolation and portability as containers.

- More automated deployment:

    In many cases, deployment involved manual steps, where developers or system administrators configured servers and 
    installed dependencies manually. This manual process often led to inconsistencies between development, testing, and production environments.

#### Docker 101

There are countless tutorials on Docker [1](https://training.play-with-docker.com/) [2](https://www.geeksforgeeks.org/docker-tutorial/), so I won't create a new one.

But for illustration purpose, here is what you can expect to see in Dockerfile. It will be useful when we will review the best practices.

```docker
# Use an official Python runtime as a base image
FROM python:3.9

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy requirements.txt to the working directory
COPY requirements.txt ./

# Install app dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application source code to the working directory
COPY . .

# Expose the application's port
EXPOSE 5000

# Define the command to run the application
CMD ["python", "app.py"]
```

## Docker best practices

In the previous section, we have seen why Docker is useful. 
Now the question is how does it work exactly and how can I make it run more efficiently.


#### Build - Optimize layers

