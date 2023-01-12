---
description: Uncover what your model really knows
tags: python logistic regression confidence interval statsmodel
img: curves_articles.png
comments: true
---

## Introduction

A lot of focus is made, in traditional machine learning in finding the right loss, the right regularization or the right architecture.

Once a model is trained, a lot of metrics exist to determine the value of the model.

On average, we know the performance will hold, but we don't know much of the failure modes. We don't know either how much the parameters would have shifted if a few data point would have moved.

These metrics seem to be as valuable as the others, so we would like to uncover the existing technics on this subject.


## 1 - Confidence interval for ML methods

#### a) Wald confidence interval for the logistic regression

Let's start with a methodology that has sufficiently large theoretical background : the logistic regression.


[This reference](https://web.stanford.edu/class/archive/stats/stats200/stats200.1172/Lecture26.pdf) gives most of the required content to explain how to compute the confidence interval of the parameters of the logisitc regression.


The set of fomulas is the following : 


$ \beta = \hat{\beta} +- z(0.05) SE(\beta) $


where beta is the parameter vector associated to the logistic regression, z is the function giving the z-value for a given confidence interval, SE stands for standard error of the beta.


The question becomes : "how do I estimate the standard error of the parameter vector" ?

The short answer : with a method called the sandwich estimate : $ Cov(\beta) = (XT \hat{W} X)^{−1}(X^T \hat{W} X)(X^T \hat{W} X)^{−1} $

Please look at [this link](https://web.stanford.edu/class/archive/stats/stats200/stats200.1172/Lecture26.pdf) if you want further details.

One of the issue of this approach is that it is model specific and might need adjustment when matrices are singular.



#### b) The bootstrap approach


##### General idea

In a situation of large sample size, we can also apply a bootstrap procedure. Elements of a training set are randomly sampled with resampling.

How does this translate, following the methodology suggested in this [blog post](). The tedious resample approach can be replaced by a wrighting using a poisson law.

The implementation is already super simple with a framework like [sk-learn](https://scikit-learn.org/stable/modules/generated/sklearn.utils.resample.html). But it can be made more efficient with setting weights of the training instead of changing the data.


##### Pseudo code example

```python
N_bootstrap = 100
X, Y = get_dataset()
models = list()

for _ in range(N_bootstrap):
  model = LogisticRegression()
  W = random.poisson(size=X.shape[0])
  models.append(model.fit(X, Y, W))

# Analyze models weights distribution
```

If this is so simple, why bother with more complicated approaches.

The resampling approach is not perfect and will have troubles to model "outlier" like data. If 1% of the data is "oulier", the resampling method will not increase this amount by a lot and the different model learned may fit this data incorrectly.

The final confidence intervals will still be less precise on this data.

The difference might become clearer when compared to the next approach : the quantile regression.



#### c) Quantile regression

The main idea of quantile regression is to learn a different set of models for each quantile of interval.

If you are interested in the [5, 50, 95] model, it will "just" 3 times more costly to have these estimates.

- Quantile regression works only for regression problem (at least the out of the box approach)
- It uses an asymetric loss dependent on the quantile q
- Any model can be used with this loss


##### Focus on the loss

The loss is defined by 

<div>
$ MAD = \frac{1}{n} \Sigma_{i=1}^n  \rho_q ( y - \beta X ) $
</div>

where $ \rho $ is defined by : 

<div>
$ \rho_q(x) = x (q - \mathbb{1}(x <= 0)) $
</div>


##### Interpretation

Let's take an example where q = 0.9.

At the start of the optimization, all samples have the same weights, because the x in the $\rho$ is approximately the same. 

Once the weights are partially learned, the samples whose $ y - \beta X $ is superior to zero are weighted with 0.9 while the others are weighted with 0.1.

The loss will only stabilize once, the most extreme point with weight 0.9 get a smaller loss. The equilibrium is found with the majority of other data point weighted at 0.1 and with a larger loss.



#### d) Summary of pros and cons of methods

We can compare the methodologies presented with this table : 


| -----------------| ----------- | ----------- | ----------- |
| Method           | Wald CI     | Bootstraps  | Quantile reg|
| -----------------| ----------- | ----------- | ----------- |
| Sample efficient | -           | --          | +           |
| Model adaptation | +/-         | +++         | -           |
| Precision        | ++          | -           | +           |


It is only a rough approximation and we wish to be more precise by studying precse usecases.

This is the goal of the next section.



## 2 - Comparison of methodologies on a few datasets

The presented approaches seems all trustworthy. But the actual results on data can be sometimes surprising.

So we test the different approaches on different well known datasets.


#### a) Iris dataset with a logistic regression

TBD


#### b) The XYZ dataset with a GBT model

TBD

