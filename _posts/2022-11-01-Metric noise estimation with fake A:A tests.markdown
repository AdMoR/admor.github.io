---
description: AN empirical method when you have doubt in your A/B test
tags: A/Btest statistics confidence interval
img: ab_test.png
comments: true
---

## Objective : 

By computing the impact of fictional A/A tests, we want to measure the natural noise occurring when creating random split on our user database (or any other split).

#### **DISCLAIMER** :  
This is not a replacement of the A/B test methodology

## Modality

#### What is the use case ? 

Let’s suppose we run an A/B test for 1 week and we get an uplift of 5% on a metric. We want to know if the result could be pure chance and come from how we did the A/B test split..

To know this results, we want to know what the uplift of random A/A tests can be.

If the data is very noisy, we could have some A/A tests reaching 4-5 % of performance. 

In that case, we would have a small risk of our test not being significant.

More precisely, 2 main parameters will change the distribution of the A/A tests : the split size (a 50/50 split is more stable than a 95/5) and the duration (it is more difficult to 
maintain an anomaly over a large duration).

The final output of the algorithm is a distribution of uplift for an A/A test.

With the previous example, we will be able to say : 1 out of 10k A/A tests matched this level of uplift.

So there is a 0.01 % chance that our result comes from pure luck.
 
#### Why would I need it on top of the traditional t-test ?

[Z-tests, Student's t-tests and Welch's t test](https://en.wikipedia.org/wiki/Student%27s_t-test) all have requirements. 

Usually a normal distribution of the metrics we follow is assumed.

But it’s difficult to know when this assumption is less true. So additional check are always welcome when the stakes are high.
This is the reason why looking at A/A tests can help us
 
#### What this test does not provide

This test is not informative about the exact uplift of the test version, because not A/B test is actually used. 

To get a similar, empirical estimation of the confidence interval, one could look at the bootstrap method (described in detail in [this post](https://www.unofficialgoogledatascience.com/2015/08/an-introduction-to-poisson-bootstrap26.html)).


## Outline 

#### Description of the steps

- Select your uids
- Create a fake a/b test table, here with 50/50 splits
- Create as many A/B test as wanted should be 1 to 10k, don’t do this one manually, it should be done programmatically with a script or a notebook.
- Compute your metric of interest per A/B test id and split
- Extract the data from the query
- Run the final analysis in a notebook
 
#### An illustrative query

The query should be run by block from a python client in order to fill the A/B test table with more than 1K A/B tests


```sql
DROP TABLE IF EXISTS  uids;
CREATE TEMPORARY TABLE uids as (
 select distinct uid
 from tmp_data.search_initiative_events

);
DROP TABLE IF EXISTS  fake_ab_test_table;
create TEMPORARY table fake_ab_test_table
as (
    select uid, 0 as abtest_id,
           STRTOL(LEFT(MD5(uid + '_' + to_char(0, '999999999999999999999')),15), 16) % 2 as split
    from uids
);


/*
-- TEMPLATE PART : where abt_id should range from 1 to 1-10k

INSERT INTO fake_ab_test_table(
 uid,  abtest_id, split,)
SELECT
    select uid, {abt_id} as abtest_id,
           STRTOL(LEFT(MD5(uid + '_' + to_char( {abt_id}, '999999999999999999999')),15), 16) % 2 as split
    from uids
*/

INSERT INTO fake_ab_test_table(
  uid, abtest_id, split)
  select uid, 1 as abtest_id,
       STRTOL(LEFT(MD5(uid + '_' + to_char(1, '999999999999999999999')),15), 16) % 2 as split
from uids;


select abtest_id, split, count(*) as cnt_event

from my_event_table
join fake_ab_test_table as abtest
on search.uid  = abtest.uid
group by abtest_id, split;
```
 
## Final analysis 
Here you run your final uplift computation per A/B test given the data you collected. One data example below : 

Result table from the previous query

 
| A/B test id | Uplift      |
| ----------- | ----------- |
| 0           | -7.0830-05  |
| 1           | -0.00284737 |


Interestingly, we already found an A/A test uplift of 0.2%, which could be the order of magnitude we could appreciate in an A/B test.
Here what should be considered is the amount of time on which the data was collected. 
 
## Conclusion 

When should you use this tool : 
- You want to estimate how noisy your data is
- You may have stopped your A/B test too early and you want to check if random chance could have provided the same uplift as your A/B test
- You want to be sure before stopping your A/B test that you are outside of the noise level of an A/A test

