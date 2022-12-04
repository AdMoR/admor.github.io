---
description: AN empirical method when you have doubt in your A/B test
tags: A/Btest statistics confidence interval
img: ab_test.png
comments: true
---

## Intro

Have you ever had doubts about some of your A/B tests ?

Did some uplift looked insanely good or impossibly negative ?

Did you ever witness an A/B test confidently saying this 10% uplit on this button color change was significant ?

If any of this interest you, we will analyse in this article why and when this can happen.


## 1 - What is the traditional process with A/B tests ?

#### a) Back to the basic 

Usually the t-test methodology is the most popular one.
The assumptions are the following : 

<div>
- Let  $$  X_1 , … , X_n $$   be independently and identically drawn from the distribution N ( μ , σ 2 )
- The quantity $${\displaystyle {\frac {{\bar {X}}-\mu }{\sigma /{\sqrt {n}}}}}$$ is a random variable whose distribution is a gaussian N(0, 1)
- We can get how far the true µ is from the computed mean given the other known quantities
</div>

This process can be understood as a posterior computation. The more samples we collect, the surer we are on the mean of the initial distribution.

The transcription of this distribution is given by the table sometimes used.

![table](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2F3.bp.blogspot.com%2F_C6375WoyYP0%2FTHVqQkJRqbI%2FAAAAAAAAASU%2FML7g-OPPgug%2Fs1600%2FStudent-t-table.png&f=1&nofb=1&ipt=2b6b3cbf78b1411c20e3d7cc1960e17cf739c9da3d72d3fdb01ee26e928d3baa&ipo=images){: width="550" }


#### b) What can go wrong : an example

Let's project ourselves in an example :

- We are on an e-commerce website selling something more popular among men than woman 
- Men convert about 90% of the time and women about 10%
- Your feature fixes a bug on the platform, you launch an A/B test to be sure everything is right
- Your A/B test finishes with -2% in conversion rate

Did you miss something on the fix ? Is your A/B test setup wrong ? Can you A/B test ?


#### b) What can go wrong : analysis 

A few things that can explain the issue in the previous setup : 

- The i.i.d. assumption means that all samples must come from the same distribution. 
  - For the event of buying, we know this is not true 
  - We lose the whole framework in this process
- A way to see what could go wrong is the following :
  - Let's suppose you got unlucky in the A/B test seeding and get 10% less mens in the test part
  - You will get 10 less booking in an iso setting, whatever the number of samples
  - The student distribution converges to low bounds when the sample size gets higher

#### c) Do I have this problem at home ?
  
**Yes and no** 

You probably don't have the same variation of conversion rate among your populations.

However, you may have a lot of different population with a lot of different conversion rates. So officially you are in this case explained before.

With lower conversion rate differences, we are closer to the i.i.d assumption. But we may be overconfident in the noise in our data.


#### d) Help me, I want to trust my data

TO COMPLETE with A/A test p values section


## 2 - Empirical approach and fictional A/B tests

If we suspect we are in a situation where we are far from the i.i.d. situation, we might be able to use an empirical approach instead of the theoretical one defined before.

#### 

By computing the impact of fictional A/A tests, we want to measure the natural noise occurring when creating random split on our user database (or any other split).

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

