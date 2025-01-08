---
title: "Generating Functions"
date: 2023-06-21T16:26:15+10:00
draft: false
math: true
---

This is another homework question from Brett Yorgey's Haskell course, 
which basically develops the ideas in https://www.cs.dartmouth.edu/~doug/powser.html.
The goal is to create a type in Haskell that can represent a formal power series
so that we can do do cool things with generating functions.

We start with a new type, which is like a cons-list but without an empty.
This is a 'stream' or infinite list.  The constructor will be `:~` because
Haskell only likes infix constructors if the first character is a `:`.
We give it the same infix precedence as regular `:` has as defined for the 
`[]` type.  This is enough for a recursive definition of a stream:

```haskell
infixr 5 :~
data Stream a = a :~ (Stream a)
```

We need some basic helpers, namely a `toList` function, a way to print a (truncated)
stream and a way to make a constant stream from a given element. We also acknowledge
 that a stream is a functor in the same way that a list is.

```haskell
streamToList :: Stream a -> [a]
streamToList (x :~ xs) = x : streamToList xs

instance Show a => Show (Stream a) where 
  show xs = show (take 8 (streamToList xs)) ++ "..."

streamRepeat :: a -> Stream a
streamRepeat x = x :~ streamRepeat x

instance Functor Stream where
    fmap f (x :~ xs) = (f x) :~ fmap f xs
```

We can specialise this to a stream of integers to represent formal power 
series with integer coefficients and define some natural numerical operations
on them, like the sum or product of two power series. 

```haskell
type Series = Stream Integer

instance Num Series where
  fromInteger n = n :~ streamRepeat 0
  negate = fmap negate
  (+) (a :~ as) (b :~ bs) = (a+b) :~ (as + bs)
  (*) (a :~ as) (b :~ bs) = (a*b) :~ fmap (*a) bs + (as * (b :~ bs))
```

We're putting these inside an instance of the `Num` typeclass, basically saying 
that a series is a type of number. This isn't really true, since we can't really 
define the sign of a power series. Because of this the compiler will complain that 
this instance of the `Num` typeclass is incomplete. We can silence that warning 
with the `{-# OPTIONS_GHC -fno-warn-missing-methods #-}` pragma though. 

The last thing to do is to define what `a/b` should mean for power series `a` and `b`.
The mathematical formulation is $\frac{\sum_{n=0}^\infty a_n x^n}{\sum_{n=0}^\infty b_n x^n} = c$ where the coeffiecients of $c$ are given by
$$
c_n = \frac{1}{b_0} \bigg( a_n - \sum_{k = 1}^n b_kc_{n-k} \bigg)
$$
which is 
```haskell
instance Fractional Series where
  (/) (a :~ as) (b :~ bs) = q where
    q = (a `div` b) :~ fmap (`div` b) (as - q*bs)
```

where again there are missing methods but that's OK.

Lastly we define a serries that represents the placeholder term (the $x$ when we 
write things like $\sum a_ix^i$)

```haskell
x = 0 :~ 1 :~ streamRepeat 0 :: Series
```

We can now do things like 
```haskell
> fibonacci = x / (1 - x - x^2)
> show fibonacci
"[0,1,1,2,3,5,8,13]..."
> squares = x * (x + 1) / (1 - x)^3
> show squares 
"[0,1,4,9,16,25,36,49]..."
```
which is an interesting but fairly crazy way to generate series.
The original lesson was supposed to be about leveraging laziness in 
Haskell but I'm more interested in the surprising aesthetics.