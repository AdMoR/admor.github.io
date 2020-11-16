---
description: A single executable and automated tests
tags: python pex tests tox
img: pex_build.png
---

This a small post presenting how to build a simple executable for your python project.


### Why do I need that ?

#### Easy launch of command line jobs

Get your executable, launch your logic with something like `my_exec -m my_package --arg1 value1`.
Job is done.

On top of that, you can store all the previous executables from earlier version of your job and run regressions tests.


### What tool to use ?

#### [pex](https://github.com/pantsbuild/pex)

It offers the packaging feature that we want.
At the end of the build, we get a \*.pex file that we can later use as `myproject.pex -m my_package --arg1 value1`.


### A boilerplate code

Available [here](https://github.com/AdMoR/pex-boilerplate), it gives the basic blocks to build a pex with your python package.

In short, run the following commands to see how your project can be built as an executable
```
tox
./dist/app.pex -m my_project
```
![pex_build]({{site.baseurl}}/assets/img/pex_build.png)

What this does is :
- Build the package requirements in a virtual env
- Install your custom package
- Create a zip that will be the .pex file
- you can launch your \_\_main\_\_.py with the second line of the command line 

