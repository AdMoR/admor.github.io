---
description: A collection of pieces of code to create useful plots
tags: matplotlib python 
---

# Code samples 

The goal of this post is to present some useful tools to plot classifcial data and some example usages.


### The cumulative distribution 

Easy to use but it took me some time to find it back.

```python
x_values_you_want_to_see = range(100)
data = [2, 2, 4, 10, 2, 2, 2, 10000]
plt.hist(data, None, histtype='step', cumulative=True, normed=True)

```

![cumulative distribution](/assets/images/cum_distribution.png)


### Plot the most frequent terms in an array

Two options : 

From the full array of values

```python
sns.countplot()
```

You have a collections.Counter built from the previous array.
It allows to display only the top-k most frequent items.

```python
def plot_count(counter,  title):
    fig, ax = plt.subplots()
    size = 10
    most_common = counter.most_common(20)
    
    for i, l in enumerate(most_common):
        rects1 = ax.bar(size * i, l[1], size / 2, label=str(l[0]))
    
    labels = [most_common[i][0] for i in range(len(most_common))]
    ticks = [size * i for i in range(len(most_common))]
    
    ax.set_title(title)
    
    plt.xticks(ticks, labels, rotation="vertical", fontsize=10)
```


# Application examples


## Basic analysis in the US accident dataset

Get the basic information

```python
import pandas as pd
df = pd.read_csv("/Users/a.morvan/Downloads/US_Accidents_May19.csv")
data = df["Weather_Condition"].values
c = Counter(data)
c_normalised = Counter({k: v / total_count for k, v in c.items()})
plot_count(c_normalised, "Accident weather type distribution")
```

![My plot count](/assets/images/distribution_weather.png)


Now let's have a look at the difference when the accident is on a roundabout

```python
df_roundabout = df[df["Roundabout"] == True]

data = df_roundabout["Weather_Condition"].values

roundabout_weather_counter = Counter(data)
total_roundabout = sum(roundabout_weather_counter.values())
center_normalised_roundabout_weather_counter = Counter({k: (v / total_roundabout) - c_normalised[k] 
                                                        for k, v in roundabout_weather_counter.items()})

plot_count(center_normalised_roundabout_weather_counter, "Accident weather type distribution on roundabouts")
```


![My plot count for subgroup](/assets/images/distribution_difference_weather.png)




