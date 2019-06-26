---
description: Does boosting works with a logistic regression ?
tags: Ada Boost torch logistic regression
---

## What is boosting ?

What is boosting ? It is the idea to use a combination of "weak" classifiers instead of a strong single one and get better performances.

$$ F(x) = \Sigma_m \gamma_m F_m(x) $$

Bossting has been made famous with GDBT (implemented in the XGBoost lib). However we are going to seen a simpler method : Ada Boost.
Ada Boost also had some glory in the past with the Viola-Jones detector that used a Haar cascade to detect face in black and white images.


#### The base function : scaled logistic regression 

We are going to work with scaled logistic regression to allow a bit more modeling power.
Here is an example of implementation

```python
import torch
import torch.nn.functional as F

class LogisticRegression(torch.nn.Module):
    def __init__(self, a):
        super(LogisticRegression, self).__init__()
        self.linear = torch.nn.Linear(1, 1)
        self.a = torch.nn.Parameter(a * torch.ones(1), requires_grad=True)
        # parameter used for the ada boost weighting between models
        self.w = torch.ones(1)
        
    def forward(self, x):
        y_pred = self.a * F.sigmoid(self.linear(x)) #+ self.b
        return y_pred
    
    
model = LogisticRegression(1.180)
```
Let's have a look at the shape of the function.
![sigmoid](/assets/images/sigmoid.png)


#### The algorithm

Now let's see how Ada Boost works.
The algorithm has many variants, but we will use a simple version. 

AdaBoost maintains a score per sample that correspond to its difficulty so far. The difficulty correspond to how much the sample is misclassified.
Each time we add a new model to the ensemble, we try to reduce the error of the model weighted by this difficulty.

$$ \epsilon_m = \Sigma_i D_m(i)[y_i \neq F(x_i)] $$

From the performance here, we will add the model to the ensemble with a coefficient based on this error.

Now, let's have a look at the implementation : 
```python
def ref_ada_boost(models, X, Y, weights=None):
	"""
	models: List[ScaledLogisticRegression]
	X : torch tensor  of shape [N, 1]
	Y : torch tensor  of shape [N]
	weights: difficulty weight of size N
	"""
	# 0 : some sanity check
    if weights is None:
        weights = (1. / X.shape[0]) * torch.ones(X.shape[0])
    else:
        weights = torch.tensor(weights)
        
    # 1. Bagging step, we sample from the dataset around half of the total size, we sample based on the difficulty
    sampling = np.random.choice(list(range(X.shape[0])), int(X.shape[0] / 2), p=weights.numpy())
    X_sampled = X[sampling]
    Y_sampled = Y[sampling]
        
    # 2. model training
    model_ = LogisticRegression(1.0)
    train_(models, model_, X_sampled, Y_sampled)
    
    # 3. computation of performance 
    pre_prediction = total_pred(models + [model_], X)
    print("Training finished with loss : {}".format(F.binary_cross_entropy(pre_prediction, Y, weight=X.shape[0] * X_importance)))
    
    prediction = (pre_prediction >= 0.5).float()
    label_correctness = (prediction == Y_sampled).float()
    
    non_agg  = weights[sampling] * label_correctness
    e = torch.sum(non_agg)
    # 3.a we compute the w to add model_ to the ensemble (models)
    model_.w = 0.5 * torch.log((1 - e) / e)
    print("w for this model", model_.w)
    
    # 3.b the recompute the weights based on performances
    sigma = 2 * (e * (1-e)) ** 0.5
    print("sigma", sigma)
    for i, index in enumerate(set(sampling)):
        weights[index] *= torch.exp(-model_.w * (1.0 if label_correctness[i] > 0 else -1.0)) / sigma
        
    weights /= torch.sum(weights)
    models.append(model_)
    return weights
        
        
my_models = list()
weights = ref_ada_boost(my_models, X, Y)

# Multiple rounds are needed to feat correctly the curve
for _ in range(10):
    weights = ref_ada_boost(my_models, X, Y, weights)
```

## Experiments

#### Feating a parabolic curve with logistic regressions

```python
import numpy as np
import random

g = lambda x: (x - 1) ** 2

f = lambda x: 1 if random.random() < g(x) else 0

x = np.array(list(map(lambda x: np.exp(x), np.linspace(-3, 0.9, 1000))))
y = list(map(f, x))

X = torch.FloatTensor(np.array(x).reshape(-1, 1))
Y = torch.FloatTensor(np.array(y))

plt.scatter(x, g(x))
```

![parabolic_curve](/assets/images/parabolic_curve.png)


##### Boosting of logistic regression
We run this boosting procedure on the described U shape.

```python
complete_prediction = sum([m.w * m(X) for m in my_new_models[:8]])
complete_prediction = torch.min(torch.ones(complete_prediction.shape), torch.max(torch.zeros(complete_prediction.shape),complete_prediction))

plt.scatter(x, complete_prediction.detach().numpy())
plt.scatter(x, g(x))

Y_pred = my_new_models[0](X).detach().numpy().flatten()
plt.plot(x, Y_pred)
```

![parabolic_curve_feated](/assets/images/parabolic_curve_feated.png)

