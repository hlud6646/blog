+++
title = 'Vectorised Operations'
date = 2024-10-09T08:01:10+11:00
draft = false
math = true
tags = ['python', 'numpy']
+++

It's well known that vectorised operations should be preferred in numpy/pandas
for reasons of performance. A less common reason is that it actually makes the 
code easier to read. For a simple example imagine that you are doing a multivariate
linear regression to transform an $n$ dimensional input to an $m$ dimensional output.
The model takes the form $y = Bx$ where $B$ is an $(n \times m)$ matrix.
Now imagine you have $k$ pairs of $(x, y)$ data, and the $k$ inputs are collected 
in an iterable data structure $X$. The naive way to gather model 
predictions is to code up a single prediction and then loop over all data:
```python
# Single observation.
y = x @ B

# All observations.
Y = np.array([x @ B for x in X])
```
Everybody will tell you this is bad because you are using slow loops, and they're
right.
But consider an alternative reason why the vectorised version would be better.
If $X$ is a $(k \times n)$ array holding your input data, you can simply write 
```python
# All observations.
Y = X @ B
```
and you will notice that apart from capitalisation the form of this equation is 
identical to the prediction for a single observation.
In other words, if your arrays have the right shape, then you can almost forget 
that you are operating on all your data at once and use a mental model of
manipulating a single observation. The vectorised version is so clear and saves
the reader having to go and examine the contents of a loop or comprehension.
