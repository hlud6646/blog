---
title: "Boolean Monoid"
date: 2022-12-07T21:04:16+11:00
draft: false
math: true
tags: ['haskell', 'algebra']
---

The homework question asks 'How many different instances of Monoid are there for the Boolean 
data type?' In stead of a mathematical proof I chose a search, motivated by the fact that there
are $16$ possible binary operators on the set $B = [True, False]$ and that I didn't want to 
check them all. I learnt some things along the way too. Starting with some definitions:
- A **Semigroup** is a type with an associative binary operator.
- A **Monoid** is a semigroup with a neutral element, i.e. a two sided inverse with respect to the operation inherited from the semigroup.

For example, the non-negative integers with the operation $+$ forms a semigroup. With the neutral element $0$ you get a monoid. The same set with $*$ (multiplication) and $1$ is another monoid.  Note that these are two different monoids defined on the same underlying type.

We are interested in Monoids on the Boolean type. One might be the logical 'and', defined in 
the source as

```haskell
-- | Boolean \"and\", lazy in the second argument
(&&)                    :: Bool -> Bool -> Bool
True  && x              =  x
False && _              =  False
```
with the neutral element being `True`. How many others are there?

First we create a type synonym for a binary operation on $B$ and an instance of `Show` for this
type so that we can print them:
```haskell
type BinaryOp = Bool -> Bool -> Bool

-- The operator (&&) is shown as
-- "  TT |-> T    TF |-> F    FT |-> F    FF |-> F  "
instance Show BinaryOp where
  show :: BinaryOp -> String
  show f = show $ concat actions where
    actions = zipWith (\x y -> concat [" ", x, " -> ", y, "  "]) ins outs
    ins     = map (\(x, y) -> show' x ++ show' y)  bools2
    outs    = map (show' . uncurry f) bools2
```

The `show` function is the messiest part of this whole thing.
For brevity we also want an alternative `show` for booleans, so we define 
```haskell
show' :: Bool -> String
show' True  = "T"
show' False = "F"
```
and for convenience some products of $B$ with itself:
```haskell
bools  :: [Bool]
bools   = [True, False]

bools2 :: [(Bool, Bool)]
bools2  = [(x, y) | x <- bools, y <- bools]

bools3 :: [(Bool, Bool, Bool)]
bools3  = [(x, y, z) | x <- bools, y <- bools, z <- bools]

bools4 :: [(Bool, Bool, Bool, Bool)]
bools4 = [(w, x, y, z) | w <- bools, x <- bools, y <- bools, z <- bools]
```

As a stepping stone to checking whether an operation is associative we first check for 
concrete arguments:
```haskell
-- Check (a . b) . c == a . (b . c) 
checkOne :: BinaryOp -> (Bool, Bool, Bool) -> Bool
checkOne f (a, b, c) = f (f a b) c == f a (f b c)
```
Associativity means that this predicate is true for any combination of a, b, c:
```haskell
associative :: BinaryOp -> Bool
associative f = all (checkOne f) bools3
```

We can create a list of all possible binary operators:
```haskell
binaryOps :: [BinaryOp]
binaryOps = do
    (a, b, c, d) <- bools4
    let f True True   = a
        f True False  = b
        f False True  = c
        f False False = d
    return f
```

and then write a predicate to check if a given boolean is a two sided inverse:
```haskell
isIdent :: Bool -> BinaryOp -> Bool
isIdent e f = and [f x e == x && f e x == x | x <- bools]
```

Finally we have a simple list comprehension to bring it all together.
```haskell
monoids :: [(BinaryOp, Bool)]
monoids = [(f, e) | f <- filter associative binaryOps,
                    e <- [True, False],
                    isIdent e f]
```

The results are
```haskell
ghci> mapM_ print monoids 
(" TT -> T   TF -> T   FT -> T   FF -> F  ",False)
(" TT -> T   TF -> F   FT -> F   FF -> T  ",True)
(" TT -> T   TF -> F   FT -> F   FF -> F  ",True)
(" TT -> F   TF -> T   FT -> T   FF -> F  ",False)
```
where we can recognize the operations as *or*, *inverse xor*, 
*and* and lastly *xor* making a total of four possible boolean monoids.
Maybe not such an interesting result but the process was fun.
