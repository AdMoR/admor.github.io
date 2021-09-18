---
description: A guide to the main features
tags: python spark mllib logistic regression
img: illu.png
comments: true
---

## Introduction

*Distributed training* is often something quite expansive to setup. Luckily with spark mllib, you can have it for a very low entry price.


This articles aims at making you gain some time in the understanding of the operators and small subtleties of the framework.



## Feature engineering

In our scenario, we assume the following : 

- We already have a dataframe containing all the information needed for the training
- However the information is not in a suitable format for a ML model


We will see an example on how to preprocess this data properly.


#### Indexing

Usually when working with tabular data, you have a lot of string and ids to process. 
Let's see the main operators to do this work.

- [StringIndexer](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.StringIndexer.html)
- [FeatureHasher](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.FeatureHasher.html)
- [VectorIndexer](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.VectorIndexer.html)

As the names are quite straightforward, I assume you already know which one you want to use.
I recommend that you look at the example given by the documentation as they are much more precise than the high level intro of the operators.


#### Bucketing
Continuous feature do not need to be directly processed but it is usually safer to do so.
This avoids cases where extreme valued samples get unreasonable predictions.

We have two main options : 
- [QuantileDiscretizer](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.QuantileDiscretizer.html)
- [Bucketizer](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.Bucketizer.html)

**QuantileDiscretizer**

Easy and safe option. The quantiles helps to find buckets that will fit the data distribution.
As the modelisation is automatic, it also help in the long term to be more responsive to data distribution changes.

**Bucketize**

This option allows you more modeling power. Data does necesarilly comes in evenly distributed buckets. So quantiles may not always be the best way to cluster some data.

However with a high number of columns, it is costly to apply a set of good thresholds.


#### Bag-of-words features

Although this may seem like an operator for text oriented problems, it can be used for other goals :
- tags available on an item
- last viewed items of a user

This operator also offers to limit the maximum voc size.  So you can easily control the size of your modelisation and avoid a million sized vector.

```python
df = spark.createDataFrame([
    (0, "12 33 123".split(" ")),
    (1, "33 12 11 123".split(" "))
], ["id", "last_ids"])

# fit a CountVectorizerModel.
cv = CountVectorizer(inputCol="last_ids", outputCol="features", vocabSize=10, minDF=2.0)
```

**Note** : You will need to transform your indexes into strings. Otherwise the CountVectorizer will start to yell at you.


#### Sparse vectorization
In order to get a sparse representation, we need to go through the vectorization of the indexes produced by previous steps. 
It is quite straighforward to use.

```
>>> df = spark.createDataFrame([(0.0,), (1.0,), (2.0,)], ["input"])
>>> ohe = OneHotEncoder()
>>> ohe.setInputCols(["input"])
OneHotEncoder...
>>> ohe.setOutputCols(["output"])
OneHotEncoder...
>>> model = ohe.fit(df)
>>> model.transform(df).head().output
SparseVector(2, {0: 1.0})
```

**Note** : Bag-of-words features are already vectorized and don't need to go through this step.


#### Cross features

Cross features is a very important part of the feature engineering step. It helps baking business logic directly inside the model.

In order to do so, the Interaction class be used as follow : 

```python
from pyspark.ml.feature import Interaction
interaction = Interaction()
interaction.setInputCols(["GENDER_vectorized", "AGE_vectorized"])
interaction.setOutputCol("GENDERxAGE")
df = interaction.transform(df)
```

Crosses made between spase vector columns are properly handled by this class. So all the crosses can be done after all feature are vectorized.


## The training 

There is less to say on this part. 
First one, you need to use a [VectorAssembler]() in order to produce the final vector used to train your model
Second one, you can access some training information with the following : 

```python
lr = LogisticRegression(maxIter=1000, regParam=0.1, labelCol="LABEL")
lrModel = lr.fit(df)
trainingSummary = blorModel.summary
print("areaUnderROC: " + str(trainingSummary.areaUnderROC))
```


## Some tricks to get some order

The main issue with this way of doing things is that you need to factor your code in order to keep your sanity.
If you have hundreds of features to process and multiple cross features to produce, doing it manually is a garanty to create bugs in your code.

#### NamedTuples to represent features

Tracking feature names can start to be difficult when you handle a few hundreds of them.
Packaging feature names into namedtuples help to store under a simple name the various subname that the feature will have through its different steps.


An example :
```python
class RawFeature(NamedTuple):

    col_name: str

    @property
    def indexed_column(self):
        return self.col_name + "_indexed"

    @property
    def vectorized_column(self):
        return self.col_name + "_vectorized"

features = {"GENDER": RawFeature("GENDER")}


indexer = StringIndexer()
indexer.setInputCols([feature.col_name for feature in features.values()])
indexer.setOutputCols([feature.indexed_column for feature in features.values()])
df = indexer.fit(df).transform(df)

# 2 - One hot encoding of all indexed features
one_hot_encoder = OneHotEncoder(dropLast=False)
one_hot_encoder.setInputCols([feature.indexed_column for feature in features.values()])
one_hot_encoder.setOutputCols([feature.vectorized_column for feature in features.values()])
df = one_hot_encoder.fit(df).transform(df)
```

Your code is now properly factored to handle dozens of similar string features !


#### Keeping the featurizer around

Contrary to the usual sklearn API, `fit` and `transform` are not in place operations.
`fit` will return a Model which will be able to execute the `transform` operation.


```python
if train:
    model = one_hot_encoder.fit(df)
    model.save("one_hot.model")
else:
    model = OneHotEncoderModel.load("one_hot.model")

df = model.transform(df)
```
This chunk of code gives the general format that must be used for all operators doing a fit_transform on the data.


#### Using the Pipeline class 

The pipeline class is very handy in order to serialize your whole process in a concise way.

Here is an example 
```python
indexer = StringIndexer()
one_hot_encoder = OneHotEncoder(dropLast=False)
my_interaction = Interaction()
assembler = VectorAssembler()
# You will need additional code in order to properly set your columns
lr = LogisticRegression(maxIter=1000, regParam=0.1, labelCol="LABEL")

pipeline = Pipeline(stages=[indexer, one_hot_encoder, my_interaction, assembler, lr])


df = pipeline.fit(df)

pipeline.save("my_pipeline.save")
```
Inference is then super easy.


