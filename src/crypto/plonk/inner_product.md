# Inner product argument

## What is it

Given the commitments of two vectors $\vec{a}$ and $\vec{b}$ of size $n$ and in some field $\mathbb{F}$, prove that their inner product $\langle \vec{a}, \vec{b} \rangle$ is equal to $z$.

Perhaps you have all the values ($\vec{a}$, $\vec{b}$ and $z$), perhaps some of them you only have commitments to.

## How is that useful

Inner products arguments are useful for several things, but what we're interested in for Mina is polynomial commitments. Specifically, we want to commit to a polynomial $f$ and then prove it's evaluation at some random point $s$. 
This is equivalent to prove that:

$$ f(s) = \langle \vec{a}, \vec{s} \rangle$$

where:

* $\vec{a} = (a_0, \cdots, a_n)$ contains the coefficients of our polynomial $f$ and 
* $\vec{x} = (1, x, x^2, \cdots, x^{n})$ contains the powers of $x$

Note that in Mina:

* $\vec{x}$ is public knowledge
* $\vec{a}$ is secret, but a commitment to it is published

Thus $f(x)$ is the resulting evaluation of the secret polynomial at a given point $x$:

$$ f(x) = a_0 + a_1 x + a_2 x^2 + \cdots + a_n x^n $$

## Settings

* The prover has two vectors $\vec{a}$ and $\vec{b}$ of size $n$.
* The prover and the verifier have pedersen hashes of the vectors $\vec{a}$ and $\vec{b}$: $\vec{a}\vec{G}$ and $\vec{b}\vec{H}$.
* They also both have the, so-far unproven, inner product of $\vec{a}$ and $\vec{b}$: $z$.
* Note that $\vec{G}$ is the vector of bases $(G_0, G_1, \cdots, G_n)$ (that have unknown discrete logarithm).
* Note that the pedersen hash of $\vec{a}$ with base $\vec{G}$ is written as $\langle \vec{a}, \vec{G} \rangle = a_0G_0 + a_1G_1 + \cdots + a_nG_n$. You can think of that as a batching of pedersen commitments, with randomness $0$ (TODO: doesn't that mean the $a_i$ have to be uniformly random? TODO2: This is a pdersen hash, not a commitment, that we explain in another section)

## The idea 

A proof is really a collection of miniproofs or subproofs.

Here, the whole idea is to reduce the size of a naive proof.
To prove the inner product involving some hidden value, we could just reveal what's in the hidden value.

$$
\boxed{\begin{aligned} 
& \langle \vec{a}, \vec{b} \rangle = z\\
& \text{given } \vec{b} \text{, } z \text{, and a hash of } \vec{a}
\end{aligned}} 

\; \overleftarrow{\text{open proof}} \; \boxed{\vec{a}}
$$

But that would be a proof in the size of the vector $\vec{a}$.

The inner product argument reduces the opening proof by using an intermediate reduction proof:

$$
\boxed{\begin{aligned} 
& \langle \vec{a}, \vec{b} \rangle = z\\
& \text{given } \vec{b} \text{, } z \text{, and a hash of } \vec{a}
\end{aligned}} 

\; \overleftarrow{\text{reduction proof}} \;

\boxed{\begin{aligned}
& \langle \vec{a'}, \vec{b'} \rangle = z'\\
& \text{ given } \vec{b'} \text{, } z' \text{, and a hash of } \vec{a'}
\end{aligned}}

\; \overleftarrow{\text{open proof}} \; \boxed{\vec{a'}}
$$

Where the size of $\vec{a'}$ is half the size of $\vec{a}$, and as such the final opening proof is half the size.

The reduction proof is where most of the magic happens, and this reduction can be applied many times to get a final opening proof of size 1! 
So most of the proof size comes from the multiple reduction subproofs that you'll end up creating on the way.

## The actual proof

TKTK

## Soundness of the reduction?

TKTK

## Adding zero-knowledgness

TKTK
