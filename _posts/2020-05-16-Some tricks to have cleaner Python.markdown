---
description: Having an easy-to-read python code is sometimes tricky
tags: python contextmanager 
---


### Usage of @property

This is something that didn't hit me originally. Why would one use a property when you can just define your variable in the `__init__` ?

One reason could be ease of reading, let's look at these two pieces of code : 


```python

class RevenueCOmputer:

    def __init__(self):
        # Imagine your data with nb_items sold per day
        self.df = pd.DataFrame([{"day": "2020-05-13", "item": 1, "revenue": 10}, 
            {"day": "2020-05-14", "item": 1, "revenue": 3}, {"day": "2020-05-13", "item": 2, "revenue": 5}])
        # Another table defines the category of the items
        self.category_df = pd.DataFrame([{"item_id": 1, "type": "flower"}, 
            {"item_id": 2, "type": "flower"}, {"item_id": 3, "type": "other"}])

    def compute_total_revenue_on_flowers(self, day):
        flower_id = self.category_df[self.category_df.type == "flower"].item_id
        return self.df[self.df.item.isin(flower_id)].where(self.df.day == day).revenue.sum()
        
```

`compute_total_revenue_on_flowers` is a bit complicated even if what is done is relatively simple.
We could separate the retrieval of the flower items to make the function simpler.

```python
    def compute_total_revenue_on_flowers(self, day):
        return self.flower_lines.where(self.df.day == day).revenue.sum()

    #######################
    #       Helpers       #
    #######################

    def find_ids_per_category(self, cat):
        return self.category_df[self.category_df.type == cat].item_id

    @property
    def flower_lines(self):
        return self.df.where(df.item.isin(self.find_ids_per_category("flower")))
    
```

With this version, we hide the complexity of manipulating the dataframe and focus on the business logic behind the function.
It seems easier to comprehend what the function does. And to verify one function at a time that everything is correct.

Of course, the more complex the function is the more gain we extract from this decomposition.


### Usage of @contextmanager

A lot of people use context manager in python whenever they do something like.

```python
with open("file.txt") as f:
    lines = f.read_lines()
```

But defining one is less usual. Let's see a silly example :

```python

@contextmanager
def read_two_files(file_a, file_b):
    with open(file_a) as f:
        with open(file_b) as g:
            yield f.read_lines() + g.read_lines()

```

The yield is what changes for contextmanager compared to the normal function. 
By using other context managers, we can keep this property of closing the files even if there is an error in the `read_lines` part.


I also liked the version where we redo the context manager from scratch using the primitives : 

```python
class DictModifier(object):
    def __init__(self, d, arg, val):
        self.d = d
        self.arg = arg
        self.val = val
        self.remember = self.d[arg]

    def __enter__(self):
        self.d[self.arg] = self.val

    def __exit__(self,*args):
        self.d[self.arg] = self.remember


d = {"a": 2}
with DictModifier(d, "a", 33) as m:
    print(d)
print(d)

"""
>>> d = {"a": 2}
>>> with DictModifier(d, "a", 33) as m:
...     print(d)
...
{'a': 33}
>>> print(d)
{'a': 2}
"""
```

I keep this example as a good reference if one day I need to do something more complex.


### Usage of namedTuple to regroup coexisting variables

Maybe this title is more enigmatic than the other. The idea is simple to understand with an example :

```python

class MyUsefulClass:

    def __init__(self, reference_path):
        # Path handling
        self.reference_path = reference_path
        self.config_path = os.path.join(reference_path, "...")
        self.temp_path = os.path.join(reference_path, "...")
        self.result_path = os.path.join(reference_path, "...")

        # Useful object definition
        self.value_a = 2
        self.seed = 33
        self.gravity = 9.81

    def do_stuff(self):
        do_thing(self.config_path, self.temp_path)

```

We have done nothing and the `__init__` is already quite packed.
We can reduce this by defining simple helper classes based on NamedTuples.

```python
class PathCollection(NamedTuple):
    reference_path: str

    @property
    def config_path(self):
        return os.path.join(reference_path, "...")

    @property
    def temp_path(self):
        return os.path.join(reference_path, "...")
```


Let's see how we can rewrite the first class

```python

class MyUsefulClass:

    def __init__(self, reference_path):
        # Path handling
        self.paths = PathCollection(reference_path)

        # Useful object definition
        self.value_a = 2
        self.seed = 33
        self.gravity = 9.81

    def do_stuff(self):
        do_thing(self.paths.config_path, self.paths.temp_path)
```

The interest of this trick is even increased if the logic on the path creation is more complex. Like creating directories or deleting them.
