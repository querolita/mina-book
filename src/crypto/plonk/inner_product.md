# Inner product argument

## What is it

Proof that $z = <\vec{a}, \vec{b}>$ in some field $\mathbb{F}$ (where $\vec{a}$ and $\vec{b}$ are of size $n$) if you have commitments to all of these values.

## How is that useful

Inner products arguments are useful for several things, but what we're interested in for Mina is polynomial commitments:

$$ f(x) = <\vec{a}, \vec{x}>$$

where $\vec{a} = (a_0, \cdots, a_n)$ contains the coefficients of our polynomial $f$ and $\vec{x} = (1, x, x^2, \cdots, x^{n})$ contains the powers of $x$.

In this setting, $\vec{x}$ is public knowledge, $\vec{a}$ is secret but a commitment is published, and $f(x)$ is the resulting evaluation of the secret polynomial at a given point $x$:

$$ f(x) = a_0 + a_1 x + a_2 x^2 + \cdots + a_n x^n $$

## Settings

Setting: 

* commitments $zG$, $\vec{a}\vec{G}$, $\vec{b}\vec{H}$
* $\vec{G}$ is a vector of bases $(G_1, \cdots, G_n)$ (that have unknown discrete logarithm).
* $\vec{a}\vec{G} = a_1G_1 + \cdots + a_nG_n$. You can think of that as a batching of pedersen commitment, with randomness $0$ (TODO: doesn't that mean the $a_i$ have to be uniformly random?)

TODO: how are these commitments hiding? Unless z, a, b totally random there's a missing randomness from pedersen.

## The idea

Idea: you can send openings to the commitments by sending the values in clear: $z, \vec{a}, \vec{b}$.
This is what we will end up doing down the line, but with shorter vectors. Right now, we assume that sending vectors $\vec{a}$ and $\vec{b}$ is expensive as they are of size $n$ and each elements are in $\mathbb{F}$.

## The protocol + proofs
 