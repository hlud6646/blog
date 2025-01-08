---
title: "Hoogle and the Rarity of Functions"
date: 2024-08-01T00:10:31+10:00
draft: false
math: true
---

>> There are only two hard things in Computer Science: cache invalidation and naming things.
— Phil Karlton

Say what you want about Haskell, *Hoogle* is an amazing tool.
If you don't know, it's simply a search engine for functions
where your search term is a function signature. That probably
sounds pretty plain. Also, my instinct was that since 
a function can do anything it likes with it's inputs so long
as the return type is correct, how useful can the signature 
alone really be? It turns out that for a given function
signature, there are usually very few commonly used functions
which match. What 
tends to happen is that the first one you see from Prelude
(the Haskell standard library) is the one you want. As a toy
example, imagine you are building a parsing library and the
first step is to identify all the pronumerals: you want the 
"x" in "420x". So you could do `dropWhile isDigit "420x"` (
  Before you get upset, I know that this is not how to build 
  a parser; it's just a demonstration :melting face:
)
I am far from an expert programmer, but I do know the `take`,
`drop`, `takeWhile` etc functions well enough, probably because
the names themselves are informative.

As the project develops, you might need to find the coefficients
 too, i.e. the "420" in this case. Since `dropWhile`
skims over that part, you might wonder if 
there is a function that can do `takeWhile` and `dropWhile` in 
one go. Well if it exists, it is a function which takes a string
and a predicate and returns two strings, which would most naturally
be returned as a tuple. If we generalise slightly 
and think of lists of a generic type rather than lists of chars (
  since that is all a string is anyway
) the the full signature must be `[a] -> (a -> Bool) -> ([a], [a])`.
If you search for this signature in hoogle you get two functions 
from Prelude. One is called `span` and does *exactly* what we need.
The other is called `break` which is almost the same, but finds the 
longest prefix where the predicate fails (rather than succeeds).
I personally find the function names `span` and `break` entirely
uninformative, but I don't blame the authors of the Prelude for
that: sometimes it is just really hard to find a good name for a 
function. That being the case, imagine how annoying it would be 
to try and find this function by any other means. Where would you 
even start? Looking at the documentation for the the list type?
Ok, you would find it eventually for sure but it is really lovely 
to be able to get it instantly just by knowing the input and return 
types. 

Finally the fact mentioned earlier about there only being a small 
number of functions for a given signature, which is what makes 
hoogle work in the first place, is an interesting one to me and
I suppose is what this post is ultimately about.
