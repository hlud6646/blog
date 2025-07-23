+++
title = '01 -  One Sample t-Test'
date = 2025-07-19T12:02:29+10:00
draft = true
math = true
+++

*Is the mean of this group what I think it is?*

**Null Hypothesis:** The population mean is $\mu_0$.  
**Alternative Hypothesis:** It is some other value.

This is an **approximate test**. The null hypothesis might be rejected more often than the specified 
significance level.

Contents:
1. t-test implementation;
1. t-test example;
1. Verification against library function;
1. Monte Carlo simulation of type I error rate;
1. Comparison of t-distribution with standard normal


## Implementation 
```python
def make_data(true_mean):
    """Return a univariate dataset of 20 samples around a given mean."""
    return np.random.normal(loc=true_mean, size=20)

def t_observed(data: np.array, mu0):
    """Calculate the observed t-statistic given observed data and hypothesised 
    value of the population mean."""
    x_bar = data.mean()
    s     = data.var(ddof=1)
    n     = len(data)
    return (x_bar - mu0) / (s**.5 / n**.5)

def plot_t_test(dof, critical_value, t_observed):
    ...

def t_test(data: np.array, proposed_mean, alpha):
    """Report the observed value of the test statistic and associated p-value for 
    the given data, significance level and hypothesised population mean."""
    t_obs    = t_observed(data, proposed_mean)
    dof      = len(data) - 1
    p_value  = 2 * stats.t.sf(np.abs(t_obs), df=dof)
    crit_val = stats.t.ppf(1 - alpha / 2, df=dof)
    return {
        "t_obs": t_obs,
        "p_value": p_value,
        "crit_val": crit_val
    }
```

## Running a Test
The next cell creates a dataset and runs the test against the hypothesis that 
the true mean is $4$. As always, we used domain specific knowledge to choose the significance 
level of $\alpha = 0.07$ and didn't automatically use $0.05$.  Since the observed value of the test statistic is not as extreme as
the critical value, we cannot reject the null hypothesis. In other words, our data do not suggest that the true
mean is different from $4$.

### How does it work?
On the plot below, the inner boundaries of the rejection region are placed so that exactly $0.07$ of the 
probability mass lies beyond them. Since the distribution is symmetrical, there must be $0.035$ of the mass 
in each tail. These boundaries define the 'critical value' of the statistic, and it is against this which we
compare the observed value. The observed value is less extreme than the critival value, and so the test does
not provide evidence that the null hypothesis is false.


Notes: 
- We do not say "Our data suggest the true mean is $4$". That's a different conclusion.
- Our test does not let us reject the null, even though we *know* it is false. Our data sampling procudure gives us 
a sample from a population centered around $4.3$.




```python
np.random.seed(123)
data = make_data(true_mean=4.3)
result = t_test(data, proposed_mean=4, alpha=0.07)
fig = plot_t_test(len(data) - 1, result["crit_val"], result["t_obs"])
plt.savefig('t-test.png', transparent=True)
result
```


    
    

{{< figure
    src="./images/01-one-sample-t-test_3_0.png"
    width="70%"
    caption="Visualisation of the two tailed t-test for the sample mean."
    >}}


    # t-test output:
    {'t_obs': np.float64(1.4738784738034707),
    'p_value': np.float64(0.15689030687628713),
    'crit_val': np.float64(1.9199915971807093)}



## Comparison With Library Function
The `scipy.stats` library provides this exact test.
We can run it and confirm that our implementation is correct.


```python
stats.ttest_1samp(data, popmean=4)
```




    TtestResult(statistic=np.float64(1.4738784738034707), pvalue=np.float64(0.15689030687628713), df=np.int64(19))



## Rejection Rate
We chose $0.07$ as the significance level in our experiment. This means that we should expect the 
test to incorrectly reject the null $7\\%$ of the time. That is, if we hypothesise the *correct* 
population mean, and redraw our sample many times, the test will tell us that the hypothesis was
not correct in $7\\%$ of the draws. 


```python
def rejection_rate():
    alpha = 0.07
    true_mean = 4
    runs = []
    for _ in range(10000):
        data = make_data(true_mean)
        test_result = stats.ttest_1samp(data, popmean=true_mean)
        runs.append(test_result.pvalue < alpha) 
    return np.mean(runs)

rejection_rate()
```

    np.float64(0.0708)


