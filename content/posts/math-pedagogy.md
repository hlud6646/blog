---
title: Some Notes on Math Pedagogy
date: 2020-04-27
description: A few reflections on teaching maths and common hurdles for early students.
math: true
tags: [math, pedagogy]
---
This is an unstructured collection of reflections on teaching and learning maths.

**Notation.**
The flexible way that notation is applied in higher studies was a challenge for me as a high school graduate.  For a long time it really did matter to me that vector quantities were written $\textbf{v}$ or $\overline{v}$ rather than $v$.
Now I couldn't care less most of the time but I think it helps to
remember that this was something that for me at least required some getting used to.

I can remember being really troubled in an early physics course that the teacher used the variable name $r$ for the radius of some component or other, and again later in the line for some other parameter.  This was too much for my loyalty to the formalism and I complained.  The professor acknowledged the issue but in hindsight if I was really following the working there wouldn't have been genuine ambiguity between a radius and the other (probably non-spatial) parameter.

For a more extreme example, I am convinced that my lecturer for topology *intentionally* wrote with a messy script to force us to focus on the underlying idea rather than the formalism.  I had the same teacher for a less advanced class later on and he was much tidier!

It's important to know when to be strict, and when to be relaxed.  If you're trying to explain quotient groups, you need to drill the idea that this group's members are
the equivalence classes of the original group.  In this case it is better to rigidly use square brackets when referring to an object on the quotient, so as not to confuse it with a representative.  These things are subtle and it's easy to make it confusing.

**Flexible vs Sloppy.**
While I appreciate all degrees of flexibility in notation, when it comes to the concepts there are some unforgivable mistakes that one hears all the way through. To say *the function $f$ of $x$* is downright sloppy, even though the phrase is ubiquitous.  In almost every example that a school or early uni student sees, $f$ is a function taking something like a number as an input, returning something like a number as its output. *$f(x)$* is then a number, not a function at all.  This most fundamental concept deserves absolute clarity.

In high-school we were given a 'proof' that  $ d/ dx \log x = 1 / x$ which involved taking the reciprocal of a $dy / dx$.  Whether or
not this is a valid in some deeper system than the myth-based calculus of year 11, it struck me as an outrageous abuse of almost meaningless placeholder symbols $dy$ and $dx$ to pretend that they were numbers and throw them around this way and a complete fluke that the result was something aesthetic.  I would say that this proof is a perfect example of something to never show to a student.  The one time that I did I felt like a fraud.

For an example of *flexible* that is as lovely as the previous example is horrible, I remember learning what it means for two groups to be *isomorphic*, something like 'the same, up to renaming of elements.'   A bunch of practice questions follow, all along the lines of
'prove that this group is isomorphic to that group.'
After a while though, the instructor's language shifted from
'blah **is isomorphic to** the Klein-4' group' to something like
'blah **is** the Klein-4 group.'  Easy to ignore the difference,
but the implication is that if two groups are isomorphic then there's really no reason to consider them different at all. The only difference is a veneer of labels, so exceedingly trivial that we must ignore the differentiation they provide.
It's fun that there is flexibility in a concept as fundamental as two things being the same.


**Notes vs Problems.**
I've seen quite a few students fall into a trap that definitely got me.  If the content is hard, it's easy to spend a few hours (days/weeks?) writing or typing gorgeous notes.  Actually its probably more like rewriting the course notes.  This is a sad waste of valuable time though compared to trying (and failing!) to solve practice problem.  If you can apply something, the content of your pretty notes is probably in your head.

**Quotient Sets.**
This is a hard concept to really take in, especially if all one is given is the abstract definition of a quotient ring or something.  I think it's worth looking at the quotient of a set by some equivalence before looking at things with more structure (groups/rings etc.)  To that end I came up with the following, somewhere in between example and analogy.

*An interior designer is interested in the  various combinations of colour which they can make from the items in a room.  To that end, they put all the  'vanilla sands' items (a rug and some wall tiles) in one corner, all the 'mocha sky' objects (a bookshelf and a bong) in another and separate the rest of the items into groups in a similar way. To compare colours, they just need to take one item from each colour group and put them next to each-other. It doesn't matter which object they choose to represent mocha-sky, so they choose a small coin painted that colour, since its easier to carry than a bookshelf.*

In this example, we started with a set of objects and an equivalence on that set, and ended up with a set of equivalence classes.
The question is, if we start with an object with a richer structure, equipped with some
equivalence, can we expect that the quotient will maintain that structure?
We would like it to go like this: A group $G$ has an equivalence relation $e$.
We denote elements of the quotient set $G/e$ by $[x]$, where $x$ is some representative
of the equivalence class.  We would like a group operation on $G/e$, which is currently nothing more than a set. A nice one would be $[x] + [y] = [x + y]$. Note that the $+$
signs mean different things.  As an operation on equivalence classes we don't know what it is yet (we're trying to define it!).  On $x$ and $y$ it is the original group operation (which OK we haven't exactly specified, but we did say that $G$ is a group and so it does have an operation.)  The idea is that if we start with two equivalence classes, we can combine them by choosing any representative, applying the original group operation, and using the equivalence class of the result.

For this to work we need to show that this operation does actually make a group
and that it does not depend on the choice of representative. The question is, 'what (if not all) kind of equivalence relation works here?'

For example,
if we have the group of integers with addition, and the equivalence
'$x == y$ if $x$ and $y$ contain the same number of 7s as digits', we have the equivalence classes $[1] = {1, 2, 11, 134 ...}$ containing all the numbers with no sevens,
$[17] = {7, 17, 70, ...}$, containing all the numbers with exactly one seven and so on.
Our operation on this quotient set would look like this:
$[7] + [7] = [7 + 7] = [14]$.  By choosing a different representative though, we can also write $[7] + [7] = [7] + [70] = [7 + 70] = [77]$.  This is broken though, since
$[14]$ does not equal $[77]$ (the class of numbers with no sevens does not equal the class of numbers with 2 sevens.)

This counterexample shows that not every equivalence relation is good enough to turn a
quotient set into a quotient group.  So what are the criteria!? TBC...

For what it's worth, I never understood this properly throughout my entire maths undergrad and it caused a fair amount of unease.  Its only recently that a friend asked for help on this topic that I really had to think about it (thank you Hughey!).


**Formula Sheets.**
I find it odd first of all that someone decided that there needs to be a 'formula' for the area of an annulus.  Surely 'take the small circle away from the big circle' is clearer than the symbolic version for students of almost any level.
This is maybe representative of a supplied formula sheet.  I would say that for a strong student, the items that are useful, e.g.  Simpson's rule for approximating the area under a curve, are mixed in with a lot of faff.  A weaker student might lose time finding the useful bit, e.g. trig-ratios, in amongst stuff they never understood, like Simpson's rule for approximating the area under a curve.

**Excerpts.**
Here is an amazing diatribe from Dieudonne's *Foundations of Modern Analysis*.
It's from the introduction to the chapter on differential calculus, and bemoans
the traditional definition of the derivative as a number rather than as a linear form.
> ... This slavish subservience to the shibboleth of numerical interpretation at any cost becomes much worse when dealing with functions of several variables: one thus arrives, for instance, at the classical formula

$D(f \circ(g_1, ...,g_n)) = \sum_{k=1}^n((D_kf) \circ (g_1, ..., g_n) \circ Dg_k) $
> giving the partial derivatives of a composite function, which has lost any trace of intuitive meaning, whereas the natural statement of the theorem is of course that the (total) derivative of a composite function is the composite of their derivatives, a very sensible formulation when one thinks in terms of linear approximations.
