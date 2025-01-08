---
title: Haskell Cheat-Sheet
date: 2022-10-22
description: ""
tags: [cheetsheet, haskell, functional-prgramming]
math: true
draft: true
---
















## Ranges
Ranges are written with an extraordinarly compact syntax, which also allows
infinite streams:
```haskell
decimalDigits = [0..9]
evens = [2,4..20]
lowerCase = ['a'..'z']
sevens = [0,7..]
sevensTable = take 12 [0,7..]
```






















## List Comprehensions
List comprehensions were designed to look like mathematical set constructions
and can contain a list of boolean guard:
```haskell
evenSquares = [ x * x | x <- [0,2..] ]
x = [ x + 1 | x <- [0,1..], predicate x ]
y = [ (x, y) | x <- [1..6], 
               y <- [1..6], 
               x + y < 10, 
               x > 1 ]
```

*Problem: Find a right triangle with integer sides and perimeter of 24.*
```haskell
head [ (x, y, z) | z <- [1..10]
                   y <- [1..z]
                   x <- [1..y],
                   x*x + y*y == z*z,
                   x + y + z == 24]
```

## Types
 Something that you might do when playing in the intepreter is `:t x`
which will return the type of your expression.

 A notable departure difference between a mathematical function and 
a programming one (even a pure one) is that the former can easily be compared
while the latter can't.  Since a mathematical function $A \to B$ is simply a 
subset of $A \times B$ which satisfies a few constraints, comparing functions 
amounts to comparing sets (EDIT: is this actually so easy though?). 
I doubt that any language would allow for the comparison of arbitrary functions.



















## Functions

### Pattern Matching Declarations
You can define functions that look like the piecewise defined funcitons 
of maths.  The patterns are matched from the top down:
```haskell
fac 0 = 1
fac n = n * fac n - 1
```

### Guards
Will the compiler warn if it suspects non-exhaustive matches?
I bet it figures partial-functions have a full right to exist.
That said you should probably desgin functions to match as many cases
as possible and so for the factorial example a version with guards would 
be better:
```haskell
fac n
  | n < 0 = 1
  | otherwise = n * fac (n -1)
```

### Let Bindings
The `let` keyword allows a declaration in a very tight scope.
Specifically in `let <bindings> in <expression>` the bindings 
defined in `<bindings>` will only be available in `<expression>`.
The whole thing is itself an expression too.
Maybe for tuple unpacking:
`let (a, b, c) = (1, 2, 3) in a + b + c`.

### Curry
All functions in Haskell take a single argument (or no arguments if that's
what variable assignment is deep down?).

### Partial Functions
means a function that is not defined everywhere, i.e. one that can raise an error.
Some people say that using these should be avoided as much as possible, which is 
surprising since `head` is such a function and one which I thought was a mainstay
of FP. Better to use pattern matching or the `Maybe` monad I guess.

### Point Free
I already had the idea that defining a function without reference to its
arguments was an aesthetic win and it turns out that this is a recognised 
style, called **point free.** 
A simple example is this:
```haskell
sumInts :: [Integer] -> Integer
sumInts = foldr (+) 0
```
































## Misc

- As in other languages the underscore symbol is used as a mapping placeholder
when we don't care about the values:
```haskell
len word = sum [ 1 | _ <- word ]
```
- To define a function intended to be used as an infix operator wrap it in brackets:
```haskell
(+-+) a b = a + b - a + b
six = 1 +-+ 3
```
- Variable assigment should be seen as declararing a zero argument function.

- It remains to be seen how common the tuple data type is, but anyway the main fact is 
that while lists are homogenous but length does not matter, tuples are heterogeneous
but have fixed length.  Note the functions `fst` and `snd` that act on pairs (tuples
of two elements) and their usefulness with `zip`.





















