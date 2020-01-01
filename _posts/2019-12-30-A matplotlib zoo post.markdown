---
description: A cheatsheet of useful plots with all the fancy options.
tags: matplotlib python 
---

# Code samples 

I collected a number of code sample to plot graphs. All of them needed me to google a few things and dive into the matplotlib documentation.
So it is usually a good starting point when you start a new analysis.


### The cumulative distribution with hist


![cumulative distribution](/assets/images/cum_distribution.jpg)


Easy enough with hist. However it is often 

```python
fig, ax = plt.subplots()
data = [np.random.lognormal(3, 0.1) for _ in range(10000)] 

max_val = int(np.quantile(data, 0.99))
min_val = int(np.quantile(data, 0.01))

ax.hist(data, None, histtype='step', cumulative=True, density=True)

ax.set_xlim(min_val, max_val)
```

You can pimp a little bit your histogram to be able to read the probabilities.
We can just change the last lines to have : 

```python
prob, buckets, _ = ax.hist(data, None, histtype='step', cumulative=True, density=True)

ax.set_xlim(min_val, max_val)
ax.set_xticks(buckets)
ax.set_yticks(prob)
```

![cumulative distribution](/assets/images/cum_distribution_with_ticks.jpg)




### Plot the most frequent terms 

Seaborn has an built-in function, but we can do a little bit better without much effort.


```python
import seaborn as sns
from collections import Counter


# Generate the data
choices = ["Poney", "Cat", "Dog", "Insect", "Bug", "Turtle", "Guinea pig", 
           "Pig", "Horse", "Lion", "Dragon", "Unicorn", "Elephant", "Others", "Kangoroo", 
           "Koala", "Dolphin", "Shark", "Rat", "Cockroach", "Beatle", "Gull", "Crow", "Eagle"]
prob = np.array([np.random.random() for _ in range(len(choices))])
prob /= sum(prob)
data = [np.random.choice(choices, p=prob) for _ in range(10000)] 

# Seaborn Countplot
fig, ax = plt.subplots()
sns.countplot(data)
plt.xticks(range(len(choices)), choices, rotation="vertical", fontsize=10)


# Custom Countplot
def plot_count(data,  title="", counted=False):
    
    plt.style.use("seaborn")
    
    if not counted:
        counter = Counter(data)
    else:
        counter = data
    
    fig, ax = plt.subplots()
    size = 10
    most_common = counter.most_common(20)
    
    for i, l in enumerate(most_common):
        rects1 = ax.bar(size * i, l[1], size / 1.5, label=str(l[0]))
    
    labels = [most_common[i][0] for i in range(len(most_common))]
    ticks = [size * i for i in range(len(most_common))]
    ax.set_title(title)
    plt.xticks(ticks, labels, rotation="vertical", fontsize=10)
    
    
plot_count(data, "Favorite animals")
```

![different countplots 1](/assets/images/seaborn_count.jpg)


![different countplots 2](/assets/images/custom_count.jpg)


With our custom plot, we can easily see which animal is everyone favorite !



### Plot the relative frequencies compared to average with custom countplots

We will use data from the US accident dataset.

```python
import pandas as pd
df = pd.read_csv("/Users/a.morvan/Downloads/US_Accidents_May19.csv")
data = df["Weather_Condition"].values
counter = Counter(data)
c_normalised = Counter({k: v / total_count for k, v in counter.items()})
plot_count(c_normalised, "Accident weather type distribution")
```

![My plot count](/assets/images/distribution_weather.jpg)


Now let's have a look at the difference when the accident is on a roundabout

```python
df_roundabout = df[df["Roundabout"] == True]

data = df_roundabout["Weather_Condition"].values

roundabout_weather_counter = Counter(data)
total_roundabout = sum(roundabout_weather_counter.values())
center_normalised_roundabout_weather_counter = Counter({k: (v / total_roundabout) - c_normalised[k] 
                                                        for k, v in roundabout_weather_counter.items()})

plot_count(center_normalised_roundabout_weather_counter, "Accident weather type distribution on roundabouts", counted=True)
```

![My plot count for subgroup](/assets/images/distribution_difference_weather.jpg)





