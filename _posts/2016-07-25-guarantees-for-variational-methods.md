---
layout: post
title: "New guarantees for approximate counting using variational methods"
#excerpt: "Interesting result from COLT 2016"
tags: [machine learning, academic]
comments: true
published: true
---

Calculating partition functions is a fundamental task in machine
learning. In general, we have a distribution $$ p $$ defined over a
domain $$ \mathcal{D} $$ specified up to normalisation, i.e. $$
p(\mathbf{x}) \propto f(\mathbf{x}) $$, $$ \mathbf{x} \in \mathcal{D}
$$, for some explicit (and usually easily computable) function $$
f(\mathbf{x}) $$. Given such a function we often want to compute the
normalisation constant, also known as the partition function, $$
\mathcal{Z} = \sum_{\mathbf{x} \in \mathcal{D}}f(\mathbf{x}) $$. This
could be because we want to compute some statistical properties of a
system, perform inference in a graphical model, or compute the
posterior probability in a latent variable model.

This problem is also well studied in theoretical computer science,
often called an approximate counting problem due to connections to
problems of combinatorial enumeration. If we consider $$ f(\mathbf{x})
$$ to be an indicator function for some set of combinatorial objects,
e.g. the set of perfect matchings of a bipartite graph, computing the
partition function is equivalent to counting the number of elements in
the set. The key early results in this field were the work of Alastair
Sinclair and his PhD supervisor Mark Jerrum, which used bounds on the
mixing times of Markov chains to give guarantees on algorithms for
approximate counting, including the partition function of the
ferromagnetic Ising model and the permanent of certain matrices
(equivalent to counting the number of perfect matchings of certain
bipartite graphs). This work was followed by a string of results,
notably one published in 2006 by Jerrum, Sinclair, and Eric Vigoda
which extended the earlier work of Jerrum and Sinclair to give a fully
polynomial time randomised approximation scheme for computing the
permanent of any non-negative matrix. The early work of Jerrum and
Sinclair, and the later work of Jerrum, Sinclair, and Vigoda, are
consider to be key results in TCS, recognised by the awards of the
Gödel Prize and Fulkerson Prize, respectively.

While Markov chains are often used in machine learning, it seems to be
far more common for people to use variational methods, which tend to
have fewer guarantees but which often converge to a solution much more
qucikly. Variational methods transform the problem of computing $$
\log \mathcal{Z} $$ into an optimisation problem which is then
approximated to find a lower bound on $$ \log \mathcal{Z}
$$. Significantly (if you are interested in this kind of thing), this
makes approximation algorithms based on variational methods
*deterministic*, whereas any approximation algorithm based on a Markov
chain will be, by definition, a randomised algorithm. To date, the
number of deterministic algorithms for approximate counting are very
few, limited to some very special cases. The problems of approximate
counting and optimisation have rich theories and have been
well-studied in theoretical computer science. Variational methods,
which could be viewed as bridging the two fields, have received less
attention and little is understood about the guarantees one can expect
from an algorithm based on variational methods.

Recently, at COLT 2016,
[Andrej Risteski](http://www.cs.princeton.edu/~risteski/) presented
some work based on quite generic ideas which provided guaranteed
approximations of $$ \log \mathcal{Z} $$ in a special case of the
Ising model. The Ising model on $$ \mathcal{D} = \{-1,+1\}^n $$ is
defined by a pairwise potential matrix $$ J $$: $$ f(\mathbf{x}) =
\exp(\sum_{i,j}J_{i,j}x_ix_j) $$. Risteski's proof is quite simple,
starting with the standard variational approximation, which can be
derived from the definition of KL-divergence (see section 3.1 of
Risteski's paper):

$$
\log \mathcal{Z}=
\max_{\mu \in \mathcal{M}}
\left(
\sum_{i,j}J_{i,j}\mathbb{E}_{\mu}[x_ix_j]
+ H(\mu)
\right)\,,
$$

where $$ \mu $$ is taken over the polytope of all distributions on $$
\{-1,+1\}^n $$. Solving this problem is intractable, but can be
approximated by relaxing the constraint that $$ \mu $$ is a
probability distribution. The _k-th element of the Sherali-Adams
Hierarchy_, $$ SA(k) $$, is defined by variables $$ \mu_S(\mathbf{x}_S)
$$ for all $$ S $$ with $$ |S| \leq k $$ satisfying the following
constraints

$$

\begin{align}

\sum_{\mathbf{x}_S}\mu_S(\mathbf{x}_S)
&= 1\,,
&|S| \leq k\\

Pr_{\mathbf{x}_S \sim \mu_S}[\mathbf{x}_{S \cap T}=\alpha] &=
Pr_{\mathbf{x}_T \sim \mu_T}[\mathbf{x}_{S \cap T}=\alpha]\,,
&|S \cup T| \leq k \end{align}

$$

Swapping $$ \mathcal{M} $$ for $$ SA(k) $$ and replacing $$ H $$ with
an appropriately defined entropy-like functional $$ \hat{H}(\mu) $$
gives a linear program, which can be solved in polynomial time to find
an upper bound on $$ \log \mathcal{Z} $$:

$$
\log \mathcal{Z} \leq
\max_{\mu \in SA(k)}
\left(
\sum_{i,j}J_{i,j}\mathbb{E}_{\mu}[x_ix_j]
+ \hat{H}(\mu)
\right)
$$

Then, mirroring approaches from combinatorial optimisation, Risteski
rounds the pseudo-distribution defined by the optimal solution $$
\mu^{\star} \in SA(k) $$ to obtain a proper distribution $$ \bar{\mu}
$$, which gives a lower bound on $$ \log \mathcal{Z} $$. Under certain
density conditions on $$ J $$, this rounding does not change the
objective value too much, and the upper and lower bounds are tight
enough to give an additive approximation for $$ \log \mathcal Z $$.

These are the very first results with a new proof technique that looks
to be reasonably general and will hopefully open the door for many
more results, much in the way the seminal work of Jerrum and Sinclair
led to fully polynomial randomised approximation schemes for a wide
variety of counting problems. The result in Risteski's paper uses some
technical results from the corresponding constraint optimisation
problem, suggesting the potential for carrying over more results from
combinatorial optimisation into approximate counting.

References
----------

1. Jerrum and Sinclair, *Approximating the Permanent*, **SIAMCOMP 18**,
   1989 [pdf](http://www.maths.qmul.ac.uk/~mj/papers/SIAMperm.pdf)
2. Jerrum and Sinclair, *Polynomial-time approximation algorithms for
   the Ising model*, **SIAMCOMP 22**, 1993 [pdf](http://www.maths.qmul.ac.uk/~mj/papers/SIAMising.pdf)
4. Jordan, Ghahramani, Jaakkola, and Saul, *An Introduction to
   Variational Methods for Graphical Models*, **Machine Learning 37**, 1999 [pdf](https://people.eecs.berkeley.edu/~jordan/papers/variational-intro.pdf)
3. Jerrum, Sinclair and Vigoda, *A polynomial-time approximation algorithm for the permanent of a matrix with non-negative entries*, **JACM 51**, 2004 [pdf](www.cc.gatech.edu/~vigoda/Permanent.pdf)
4. Riteski, *How to calculate partition functions using convex
programming hierarchies: provable bounds for variational methods*,
**COLT 2016**
[pdf](http://www.jmlr.org/proceedings/papers/v49/risteski16.pdf)
