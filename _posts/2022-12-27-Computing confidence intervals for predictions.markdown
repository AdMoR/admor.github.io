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

#### a) Hessian based confidence interval


If you are maximising a likelihood then the covariance matrix of the estimates is (asymptotically) the inverse of the negative of the Hessian. 
The standard errors are the square roots of the diagonal elements of the covariance ([1](https://stats.stackexchange.com/questions/27033/in-r-given-an-output-from-optim-with-a-hessian-matrix-how-to-calculate-paramet), [2](https://stats.stackexchange.com/questions/68080/basic-question-about-fisher-information-matrix-and-relationship-to-hessian-and-s)). 


[This reference](https://web.stanford.edu/class/archive/stats/stats200/stats200.1172/Lecture26.pdf) gives most of the required content to explain how to compute the confidence interval of the parameters of the logisitc regression.


The set of fomulas is the following : 


$ \beta = \hat{\beta} +- z(0.05) SE(\beta) $


where beta is the parameter vector associated to the logistic regression, z is the function giving the z-value for a given confidence interval, SE stands for standard error of the beta.



#### b) The bootstrap approach


In a situation of large sample size, we can also apply a bootstrap procedure. Elements of a training set are randomly sampled with resampling.

How does this translate, following the methodology suggested in this [blog post](https://www.unofficialgoogledatascience.com/2015/08/an-introduction-to-poisson-bootstrap26.html). The tedious resample approach can be replaced by a wrighting using a poisson law.

The implementation is already super simple with a framework like [sk-learn](https://scikit-learn.org/stable/modules/generated/sklearn.utils.resample.html). But it can be made more efficient with setting weights of the training instead of changing the data.



##### Pseudo code example

The priinciple is easy to illustrate. We can use the weight parameter of the fit method of sklearn model.

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

If this is so simple, why bother with more complicated approaches :p .



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

We can compare the methodologies presented with these tables : 


![Pros 1]({{site.baseurl}}/assets/img/pros_1.png){: width="500" }
![Pros 2]({{site.baseurl}}/assets/img/pros_2.png){: width="500" }
![Pros 3]({{site.baseurl}}/assets/img/pros_3.png){: width="500" }


It is only a rough summary. We will see more details with examples.



## 2 - Comparison of methodologies on a few datasets

The presented approaches seems all trustworthy. But the actual results on data can be sometimes surprising.

So we test the different approaches on different well known datasets.


#### a) The affair dataset

Available on [this page](https://www.statsmodels.org/stable/datasets/generated/fair.html) 

It looks like the following 

![Affair]({{site.baseurl}}/assets/img/affair_dataset.png){: width="500" }

We first trained on this dataset a logistic regression using technic 1 and 2 : Hesian based and bootstraps.

We compare the parameter confidence interval of two method on the first variable.

On this first figure, for each model, we extracted the weight corresponding to the first variable and made an histogram.


![Boostrap histogram]({{site.baseurl}}/assets/img/boostrap_weigh_hist.png){: width="500" }

The array of the weights collected from all models allows to have a simple estimation of confidence interval by taking the nth percentile in the sorted array.


Stats model directly gives us the CI of a paramter when looking at the summary of the model.

![statsmodels ci]({{site.baseurl}}/assets/img/stats_model_ci.png){: width="500" }


We can then compare the values found by the two methods : 

![Comparaison of CI]({{site.baseurl}}/assets/img/comparaison_of_ci.png){: width="500" }


To my own surprise the two methods have approximately the same results. This is a really good results for bootstrap as one might expect less from an empirical method.



#### b) The XYZ dataset with a GBT model

TBD

