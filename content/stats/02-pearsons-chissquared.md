+++
title = 'Pearsons Chis-squared Test for Independence'
date = 2025-07-22T20:45:15+10:00
draft = true
math = true
+++

# Pearson's $\chi^2$ Test
*Are green eyed and brown eyed people equally likely to be vegetarian?*

**Null Hypothesis:** Vegetarianism occurs at the same rate among green eyed people and brown eyed people.  
**Alternative hypothesis**: Vegetarianism is related to eye color.

Contents:
1. Experimental setup
1. 


```python
import scipy.stats as stats
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from tqdm.notebook import tqdm
sns.set_palette("muted")

# Fix experimental parameters:
N_GREEN = 70
N_BROWN = 50
N = N_GREEN + N_BROWN
ALPHA = 0.042
```

Define a reusable sampler and a convenience methods for displaying contingency tables.


```python
def make_data():
    a = stats.binom.rvs(N_GREEN, .3)
    b = N_GREEN -  a
    c = stats.binom.rvs(N_BROWN, .3)
    d = N_BROWN -  c
    return np.array(((a, c), (b, d)))

def to_pandas(arr):
    """Wrap a contingency table in a Pandas array."""
    return pd.DataFrame(
        arr, 
        columns = pd.Series(["Green Eyes", "Brown Eyes"], name="Eye Color"),
        index = pd.Series(["Vegetarian", "Carnivour"], name="Eating Habit")
    )

def plot_contingency_table(df, ax, **kwargs):
    """Create a bar plot of the contingency table."""
    df = df.stack().reset_index()
    df.columns = list(df.columns[:2]) + ["Count"]
    ax = sns.barplot(df, hue="Eating Habit", y="Count", x="Eye Color", alpha=.8, ax=ax)
    ax.set(**kwargs)
    return ax
```

## Implementing the Test
This test is pretty simple mathematically. You build the test statistic from the observed frequencies
and the expected frequencies under the null hypothesis.


```python
def get_expected_frequencies(observed):
    row_sums = observed.sum(axis=1)[:, np.newaxis]
    col_sums = observed.sum(axis=0)
    return row_sums * col_sums / N

def test(observed):
    expected = get_expected_frequencies(observed)
    observed_statistic = ((observed - expected) ** 2 / expected).sum()
    p_value = stats.chi2.sf(observed_statistic, df=1)
    return observed_statistic, p_value
```

## Running a Test
We note that our test agrees with the library implementation.


```python
np.random.seed(123)
observed = make_data()
print(test(observed))
stats.chi2_contingency(observed, correction=False)
```

    (np.float64(0.653061224489796), np.float64(0.41902033353431567))





    Chi2ContingencyResult(statistic=np.float64(0.653061224489796), pvalue=np.float64(0.41902033353431567), dof=1, expected_freq=array([[21., 15.],
           [49., 35.]]))



## Rejection Rate
We find that the test incorrectly rejects the null at a rate close to the predetermined $\alpha$ level.


```python
def rejection_rate():
    runs = []
    for _ in tqdm(range(10**4)):
        observed = make_data()
        _, p_value = test(observed)
        runs.append(p_value <= ALPHA)
    return np.mean(runs)

rejection_rate()
```


      0%|          | 0/10000 [00:00<?, ?it/s]





    np.float64(0.0387)




```python

```


```python

```


```python

```
