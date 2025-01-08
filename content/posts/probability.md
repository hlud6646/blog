---
title: A First Look at Rigorous Probability Theory (WIP)
date: 2023-06-19
description: "nice description"
tags: [math, probability, measure-theory, homework]
math: true
draft: true
---

This post contains solutions to the exercises to Rosenthal's book.
Probably not interesting to anyone but me.










## 1. The Need For Measure Theory

### 1.3.2
_Suppose $\Omega = \{1, 2, 3\}$ and $F$ is its powerset.
Find necessary and sufficient conditions on the numbers
$x = P\\{1, 2\\},\  y = P\\{2, 3\\}, \ z = P\\{1, 3\\}$
which make $P$ a countably additive probability measure._

If $P$ is assumed to be a probability measure (hence countably additive) then
$P\\{1\\} + P\\{2\\} = x$ with similar relations for $y$ and $z$.
Then

$$2 = 2(P\\{1\\} + P\\{2\\} + P\\{3\\}) = x + y + z  \ \ \ \ \  (*)$$

Obviously we will also require that
$P\\{\emptyset\\} = 0$ and $P\Omega = 1$, from which we get
$P\\{1\\} = 1 - P\\{1\\}^C = 1 - y$ with similar relations for the other outcomes:

$$
\begin{aligned}
  &P\\{1\\} = 1 - y,\\\
  &P\\{2\\} = 1 - z,\\\
  &P\\{3\\} = 1 - x (**)
\end{aligned}
$$

To verify that $(*)$ and $(**)$ in turn define a proper measure
(i.e. that they are also sufficient conditions) assume these relations
and verify that $P$ is additive for any pair of disjoint subsets.
For example,
$$
P\\{1\\} + P\\{2\\}  \  = \
1 - y + 1 - z = 2 - (x + y + z) + x \ = \
x = P\\{1, 2\\}
$$
as required.
























### 1.3.3
_Define $P$ on sets of natural numbers by $P(A)$ = $0$ if $A$ is finite and $1$ if infinte. Is $P$ finitely additive?_
No, since we would then have
$$
1 = P(\mathbb{N}) = P(odds) + P(evens) = 2.
$$































### 1.3.4
_Let $P$ be the counting measure on $\mathbb{N}$, that is $|A|$ if $A$ is finite or $\infty$ otherwise.
Is $P$ finitely/countably additive?_
If $A_i$ is a finite and disjoint collection of natural numbers then
$|\cup A_i| = \sum |A_i|$, i.e. $P$ is additive.
If $A_i$ is a countably infinite and disjoint collection then each set
contains at least one element. Then
$\sum_\mathbb{N}|A_i| \geq \sum_\mathbb{N} 1 = \infty$ and
$|\cup A_i|$ must also contain an infinite number of elements.



















<!-- ### 1.3.5 -->
<!-- _(b) Exhibit a function on $2^{[0,1]}$ which is countably additive -->
<!-- and shift invariant but not defined on intervals as length._ -->
<!-- Let $f(A)$ be $0$ if $A$ is countable and $1$ otherwise. This funnction is -->
<!-- clearly shift invariant and does not represent lengths. The fact that $f$ -->
<!-- is countably additive follows from the result that says "a countable union -->
<!-- of countable sets is countable."  In detail, if $\{A_i\}$ is some family of -->
<!-- disjoint sets indexed over $\mathbb{N}$ then either no single $A_i$ is uncountably -->
<!-- infinite or there is at least one which is. In the first case -->
<!-- $\sum P(A_i) = \sum 0 = 0$ -->
<!-- while by the result just claimed $P(\cup A_i) = 0$ since the union is countable. -->
<!-- In the other case both $\sum P(A_i)$ and $P(\cup A_i)$ are $1$ -->




































## 2. Probability Triples

### 2.2.3
_Let $J$ be the collection of all intervals in $[0,1]$. Then $J$ is a semi-algebra._
By definition we include degenrate intervals, so $J$ contains the empty set.
Also the full space $[0,1]$ is an interval and so also in $J$.
The intersection of two intervals is again an interval and so by induction is any
finite intersection of intervals. For various kinds of intervals we note that
the complement is a disjoint union of other intervals.  (Note that this "union" might
be of an interval and the empty set, and that it might not be unique.  Both kinds appear
in the list below.)
$$
\begin{aligned}
  &\\{\\}^C = [0, 1] \\\
  &\\{0\\}^C = (0, 1] \\\
  &\\{ a\\}^C = [0, a) \dot{\cup} (a, 1],\ \  a > 0\\\
  &(a, 1)^C = [0, a] \dot{\cup} (a, 1) \ or \  [0, a) \dot \cup [a, 1], \ \ a > 0 \\\
  &(a, b]^C = [0, a] \dot \cup (b, 1]
\end{aligned}
$$


























### 2.2.5
_Let $B_0$ be the collection of all fininte unions of elements of $J$. Show that $J$ is an
algebra but not a $\sigma$-algebra._
By the previous exercise and De Morgan's laws $J$ is closed under taking complements, finite unions and finite intersections, i.e. is an algebra of sets. On the other hand if it were a $\sigma$-algebra then it would contain countable unions of its elements, but by defenition these would also have to be finite unions.























### 2.3.16
_Let $(\Omega, \mathcal{M}, P*)$ be the probability triple provided by the extension theorem applied to some
semi-algebra of sets $J$ with well behaved (as defined in 2.3.1) set function $P$. Then $P*$ is complete._
Let $A$ be a set in $\mathcal{M}$ with zero measure and suppose $B \subseteq A$.
Using the decomposition $A \setminus B \ \dot \cup \  A^C = B^C$ we have
$$P*(B \cap E ) + P*(B^C \cap E) = P*(B \cap E) + P*((A \setminus B) \cap E) + P*(A^C \cap E).$$
The first two terms on the right vanish, leaving only
$P*(A^C \cap E)$ which we know is less than $P*(E)$ by the membership of $A$ in $\mathcal{M}$. Hence $B$
is in $\mathcal{M}$ also.






























### 2.4.3
This problem verifies countable monotonicity for the length function on
real intervals.


_(a) Let $A_1, ... A_n$ be a finite colletion of intervals and $I$ some interval with
$I \subseteq \cup A_i$.  If $P$ describes the length of an interval then
show that $P(I) \leq \sum P(A_i)$._

Reorder the $A_i$ so that their left endpoints form an increasing sequence
$a_i$. Define $B_i = [a_i, a_i+1)$ for $i = 1, ..., n - 2$ and $B_n = A_n$
and note that $B_i$ is a disjoint family whose union conincides with the union
of the $A_i$. Then by monotonicity and the result of 2.4.2
$$
  \sum P(A_i) \geq \sum P(B_i) = P(\cup B_i) = P(\cup A_i).
$$
Note the possibility of an annoying case where $A_{n-1}$ is of the
form $[a, b]$ and $A_n$ is $(b, c]$.  In our construction the element
$b$ will be missed and so $A_n$ should be redefined as $[b, c]$.


_(b) Let $I_i$ be a countable collection of open intervals and $I$ some closed interval
with $I \subseteq \cup I_i.$ Then $P(I) \leq \sum P(I_i)$._
By the Heine Borel Theorem the open cover $I_i$ of $I$ admits a finite subcover,
$I_1, ..., I_n$ (after renaming).
The previous result and the fact that $P$ takes its values
in the non-negative reals imply
$$
P(I) \leq \sum_{i=1}^n P(I_i) \leq \sum_\mathbb{N}P(I_i).
$$
_(c) Finally let $I_i$ be any countable collection of intervals
and $I$ any interval with $I \subseteq \cup I_i$. Then
$P(I) \leq \sum P(I_i)$._
For each $I_i$ with endpoints $a_i, b_i$ define the open interval
$J_i = (a_i - 2^{-i}\epsilon, b_i + 2^{-i}\epsilon)$, that is the original
interval made open and extended by a small amount. Each $J_i$ contains its
corresponding $I_i$, and the whole collection forms an open cover of $\overline{I}$
(the closure of $I$). The previous result yields
$$
P(\overline{I}) \leq \sum P(J_i).
$$
Noting that $P(J_i) = P(\text{int}(I_i)) + 2^{1 - i}\epsilon$
and that an interval and its interior/closure have the same length we have
$$
P(I) = \leq \sum P(I_i) + \sum 2^{1-i}\epsilon
$$
where the last term equals $2\epsilon$ and can be made as small as we like.































### 2.4.5
_Let $A$ be the collection of $(\infty, x]$ for real $x$
Then the $\sigma$-algebra generated by $A$ is the Borel
$\sigma$-algebra on $\mathbb{R}$._

The function that takes a set and returns the $\sigma$ algebra
generated by that set is clearly monotonic.  Hence
$\sigma(A) \subset \sigma(\\{\text{all intervals}\\}) = B$
and to prove equality we only need to show the reverse inclusion.



Let $a$ be some real number.
$\sigma(A)$ contains the half open intervals
$$(-\infty, a + 1/n] \cup (-\infty, a - 1 / n]^C = (a - 1/n, a + 1/n]$$
and hence their intersection over $\mathbb{N}$, that is $\\{a\\}$.

Similarly for any $a < b$ we can write
$$(-\infty, a] \cup (-\infty, b]^C = (a, b]$$ and so taking unions or
intersections with the singleton sets described above we find that
$\sigma(A)$ contains all intervals. But $B$ is defined as the smallest
$\sigma$-algebra which contains all intervals, so $B \subseteq \sigma(A)$
as required.































### 2.4.7
_(a) Prove that the Cantor Set and its complement are Borel Sets._
The Cantor Set $K$ is constructed from the unit interval by iteratively removing
a finite number of line segments. At the first stage one segment is removed,
at the $n$-th stage $2^{n-1}$ segments are removed. Removing a line segment is
nothing more than taking an intersecion with the complement of that segment (i.e.
interval) and as this procudure is repeated over $\mathbb{N}$ we have that $K$ is a Borel set.

Complement?

_(b) Prove that the Cantor Set and its complement are Lebesgue Measureable._
Is this not as simple as pointing out that $B \subseteq M$?

_(c,d,e) Let $B_1$ be the set containing all finite or countable unions of intervals in $[0,1]$. This
set does not form an algebra._

The inductive process of removing intervals from the unit interval to end up with the Cantor Set can be seen from another
angle as adding intervals to the empty set to end up with its complement.
In this light it is clear that $K^C$ is a member of $B_1$.

First observe the fact that if a set contains a non-singleton closed interval then it contains an open interval.
Now the Cantor Set cannot contain an open interval, since the inductive
construction ensures that this interval would eventually be chopped. By the (contrapositive of) the previous remark
if $K$ contains a closed interval then that interval must be a singleton. If however $K$ could be written
as the countable union of such intervals then it would itself be countable, which it is not. Hence
$K \nsubseteq B_1$.

By the previous results $B_1$ is not closed under taking complements and is therefore not a $\sigma$-algebra.


































<!-- ### 2.5.6 -->





### 2.6.1 (a)
_(A semi-algebra for infinite coin tossing.)_
The intersection of two sets $A_{\alpha_1...\alpha_n}$ and $A_{\beta_1 ... \beta_n \beta_{n+1}... \beta_m}$
is either empty if not every $\alpha_i = \beta_i$ for $1...n$ else it equals the second one.
Hence $J$ is closed under finite intersections.
If we write $\alpha_i'$ for the negation of $\alpha$ (i.e. $0' = 1, 1' = 0$) then we can write the complement
of $A_{\alpha_1...\alpha_n}$ as
$$
A_{\alpha_1'} \cup A_{\alpha_1 \alpha_2'} ... A_{\alpha_1 \alpha_2 ...\alpha_{n-1} \alpha_n'}
$$
which is a disjoint union of sets in $J$.




























### 2.6.4
_Verify that $J = \\{A \times B, A \in F_1, B \in F_2\\}$ is a semi-algebra._
For sets $A, C$ in $F_1$ and $B, D$ in $F_2$ we have
$A \times B \cap C \times D = (A \cap C) \times (B \cap D)$.
The two intsersections are in $F_1, F_2$ respectively (since those are
$\sigma$-algebras and so the product is in $J$ by definition.

By a similar argument it is easy to see that $(A \times B)^C = A^C \times B^C$ which
is a finite disjoint union of members of $J$ (in fact a degenerate union).
































### 2.7.4
_Let $F_1 \subseteq F2,...$ be an increasing sequence of collections of subsets
of $\Omega$._

_(a) Suppose that each $F_i$ is an algebra.
Then $\cup _ \mathbb{N} F_i$ is an algebra._
Let $A_1...A_n$ be some finite collection of sets in $\cup_\mathbb{N} F_i$.
Then each $A_i$ is contained in $F_{s(i)}$ for some index $s(i)$.
Since $F_i$ is an increasing sequence the largest of the $F_{s(i)}$
actually contains all the $A_1...A_n$, and also contains their union
(since every $F_i$ is an algebra).

_(b) The same cannot be said for $\sigma$-algebras._































### 2.7.5
_Let $F$ be the collection of subsets of $\mathbb{N}$ such that
either $A$ or $A^C$ is infinite._

_(a) $F$ is an algebra._
Let $A, B$ be members of $F$. If both are finite then so is $A \cup B$.
Furthermore $(A \cup B)^C$ is $\mathbb{N}$ with a finite set removed,
which is infinte.  If one (say $A$) is infinite then
$A \cup B$ is infinite, and $(A \cup B)^C$ is contained in
$A^C$ which is finite.

_(b) $F$ is not a $\sigma$-algebra._
Conisder the sequence of singleton sets $\\{2\\}, \\{4\\}$ etc. over the even
numbers.  Each is in $F$ since each is finite and obviously has an infinite complement
but their union (the even numbers) is not.

_(c) Define $P$ on $F$ by $P = 0, 1$ on finite and infinite sets respectively.
Is $P$ finitely additive?_ If so we would have
$P(odds \cup evens) = P(odds) + P(evens)$.

_(d) Is $P$ (as defined above) countably additive?_
Let $A_n$ be a countably infinite collection of disjoint sets.
Furthremore, suppose that they are all finite. Then
$\sum_\mathbb{N} P(A_i) = \sum_\mathbb{N} 0 = 0$. But each
set contains at least one element and so their
their (disjoint) union over $\mathbb{N}$ must be infinite, i.e. has measure $1$.
**[Fine but show me such a collection!]**



























### 2.7.6
_Let $F$ be the collection of subsets of the unit interval with the property that
if $A$ is in $F$ then either $A$ or $A^C$ is finite. Let $P(A)$ be $0$ if
$A$ is finite and $1$ if $A^C$ is finite._

(a) $F$ is an algebra. It is immediate that $F$ contains the empty set, the full unit
interval and that $F$ is closed under taking complements. To check that it contains
finite unions first assume that $A, B$ are both finite. Then so is their union, and the
complement of their union is infinte. Else if $B$ is infinite, then $A \cup B$ is infinite.
We require that $(A \cup B)^C$ is finite, and by De Morgan's Laws we have that this set is
not larger than $B^C$, which is finite.

(b) $F$ is not a $\sigma$-algebra. Consider the countable union of singleton sets of rational numbers.
The result is $[0, 1] \cap \mathbb{Q}$, which is not contained in $F$.

(c) If $A$ and $B$ are two finite sets then so is their union and
$P(A \cup B) = 0 = 0 + 0 = P(A) + P(B).$
Otherwise if $B$ is infinite then so is the union with $A$ and
$P(A \cup B) = 0 + 1 = P(A) + P(B)$.
Lastly if we can find two disjoint and infinite members of $J$ then we will see that $P$
fails to be additive, since we would have $P(A \cup B) = 1 \neq 2 = P(A) + P(B)$.
This turns out to be impossible by the following.
We have that $A \cap B = \emptyset$ and so $(A \cap B)^C = A^C \cup B^C = [0,1]$.
But $A^C$ and $B^C$ are both finite so their union can't possibly be the full unit interval.
Hence $P$ is finitely additive.

(d) By the previous part it is not possible to have a disjoint union of two infinite members in
$F$.  So to test for countable additivity we only need to check whether $P$ is additive on a
countably infinite collection of finite sets. This construction turns out to be impossible and
so $P$ is countably infinite, although only by vacuous truth.

The reasoning is that if $A_i$ is a family of finite sets indexed over $\mathbb{N}$ then their
union is a countably infinite set, whose complement is therefore uncountably infinite.  This
precludes the union from membership in $F$ and so we don't even need to check the additivity of
$P$ on this set.




































### 2.7.7
_Let $F$ be the collection of all subsets of the unit interval such that
either $A$ or $A^C$ is countable.
Define $P = 0, 1$ on countable, uncountable sets respectively._

_(a) $F$ is an algebra._ Suppose $A, B$ are two sets in $F$.
If both are countable then $A \cup B$ is also countable and $(A \cup B)^C$ is the unit
interval with a countable set removed, which is uncountable.
Otherwise if $A$ is uncountable then so is $A \cup B$,
while $(A \cup B)^C = A^C \cap B^C$ is smaller than $A^c$ which is countable.

_(b) $F$ is a $\sigma$-algebra._
Let $A_1,A_2,...$ be a collection of sets in $F$.
Suppose firstly that all are countable. Observe that a countable
union of countable sets is countable, and that the unit interval less
a countable set is uncountable.
Otherwise at least one of the $A_n$ (call it $A_1$) is uncountable, and then so is their union.
Also $(\cup_\mathbb{N}A_i)^C$ is contained in $A_1^C$ which is countable, hence so
is the entire union.

<!-- _(c) P is not finitely additive._ -->
<!-- We look for two disjoint uncountable subsets $A, B$ of $[0,1]$, since we could then have -->
<!-- $P(A \cup B) = 1$ and $P(A) + P (B) = 2$.  The Cantor Set and its complement will do. -->
<!-- Nooooo! The cantor set is not in F. -->













<!-- ### 2.7.9 -->
<!-- _The cardinality of a finite $\sigma$-algebra is a power of 2._ -->
<!-- Consider the non-empty sets $B_1...B_m$ in $\Sigma$ that have no proper subsets. -->
<!-- Notice that if they were not disjoint, then $B_1, B_2$ would have some common element -->
<!-- and $B_1 \cap B_2$ would be a proper subset of each, which is not possible. -->




<!-- ### 2.7.10 -->
<!-- This is a simple calculation. -->


### 2.7.11
_In constructing the Borel $\sigma$-algebra of subsets of $[0,1]$ you could also just start with
semi-open intervals._
Let $J'$ be the collection of all half open intervals $(a, b]$, together with
$\\{0\\}, \emptyset$ and $[0,1]$.
Let $J$ be the collection of all intervals.
First note that $\sigma(J')$ contains all the
singletons, since for any $b > 0$ we have
$$
\\{b\\} = \cap_\mathbb{N} (b - 1 / n, b].
$$
Using these singletons $\sigma(J')$ also contains all intervals.
Since $\sigma(J)$ is the smallest $\sigma$-algebra with this property we have
$\sigma(J) \subseteq \sigma(J')$.
But $J'$ is smaller than $J$ which gives the reverse inclusion.




















### 2.7.12
Let $D_n = K \oplus\  1/n$. By shift invariance $\lambda(D_n) = 0$ for every $n$.
If you draw it it looks like $\cup_\mathbb{N} D_n$ is a really large set but still
$$
\lambda( \cup_\mathbb{N} D_n) \leq \sum_\mathbb{N} \lambda(D_n) = \sum_\mathbb{N} 0 = 0.
$$









































### 2.7.13
_For a sample space $\Omega$, semi algebra $J$, find a non-negative function $P$ with
$P(\emptyset) = 0$ and $P(\Omega) = 1$ which fails to be countably additive._
Let the sample space be the non-negative real numbers and the semi-algebra be the set of all
intervals. Then we can define $P(\Omega) = 1$ and $P = 0$ for any other set.
We also have
$$
P \bigg( \cup_\mathbb{N} [n, n+1)\bigg) = 1 \neq 0 = \sum_\mathbb{N} P[n, n+1).
$$






























### 2.7.14
(This one is easy calculating, and noticing that $J$ is not a semi-algebra since
it fails to contain intersections.)



















<!-- ### 2.7.15 -->
<!-- wtf -->































### 2.7.16
The hypothesis of countable monotonicity on $P$ is used to prove
countable monotonicity on $P*$, which is used twice in the proof 
of the extension theorem: to show that $P*$ is an extension of $P$
and to show that the $\sigma$-algebra $M$ contains $J$.

<!-- and how would the conclusions be altered?? -->







































### 2.7.17
Let $\Omega = \\{1, 2\\}$ and $J$ be all subsets. Define $P$ by 
$P\emptyset = 0, P1 = P2 = 1/3, P12 = 1$.

$J$ contains all intersections and complements, hence is a semi-algebra.
To check finite superadditivity of $P$ we only need to check that 
$P(1 \cup 2) \geq P1 + P2$ which does hold.
However $P$ does not posses countable monotonicity, as
we have $1, 2 \in J$, $12 \subseteq 1 \cup 2$ but not
$P(12) \leq P1 + P2.$

<!-- and the last part -->































### 2.7.19
*$J$ is a semi-algebra.*
The intersection of any two sets in $J$ is one of the following:
$$
\begin{aligned}
  &\emptyset \cap \\_  = \emptyset\\\
  &n \cap m = \emptyset\\\
  &n \cap \Omega = n
\end{aligned}
$$
from which $J$ is closed under intersection.
Furthermore the complement of any singleton is the finite disjoint of all
other singletons.

*Finite superadditivity on $P$.*
This holds trivially with equality, since the only finite disjoint family of elements in $J$
whose union is also in $J$ is the union of all the singletons.
In that case we have $P(\cup_\Omega \\{ \omega_i \\}) = P(\Omega) = 1$
and $\sum_\Omega P(\\{\omega_i\\}) = \sum_\Omega p(\omega_i)$ which is also 
$1$ by definition.

*Countable Monotonicity on $P$*.
Every element $\omega$ is contained in exactly one element of $J$ other 
than the full space. So if $\\{\omega_i\\}$ is some collection of singletons 
with $\\{\omega\\} \subseteq \cup \\{\omega_i\\}$ then we must have 
$\omega = \omega_i$ for one of the $\omega_i$.
As $P$ is non-negative it follows that $P(\\{\omega\\}) \leq \sum P(\\{\omega_i\\})$.
