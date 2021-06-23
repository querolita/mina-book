# Inner product argument

## What is it

Given the pedersen hashes of two vectors $\vec{a}$ and $\vec{b}$ of size $n$ and in some field $\mathbb{F}$, proove that their inner product $\langle \vec{a}, \vec{b} \rangle$ is equal to $z$.

Perhaps you have all the values ($\vec{a}$, $\vec{b}$ and $z$), perhaps some of them you only have commitments to.

## How is that useful

Inner products arguments are useful for several things, but what we're interested in for Mina is polynomial commitments:

$$ f(x) = \langle \vec{a}, \vec{x} \rangle$$

where $\vec{a} = (a_0, \cdots, a_n)$ contains the coefficients of our polynomial $f$ and $\vec{x} = (1, x, x^2, \cdots, x^{n})$ contains the powers of $x$.

In this setting, $\vec{x}$ is public knowledge, $\vec{a}$ is secret but a pedersen hash is published (TODO: isn't it bad that it's not a commitment instead?), and $f(x)$ is the resulting evaluation of the secret polynomial at a given point $x$:

$$ f(x) = a_0 + a_1 x + a_2 x^2 + \cdots + a_n x^n $$

## Settings

* The prover has two vectors $\vec{a}$ and $\vec{b}$ of size $n$.
* The prover and the verifier have pedersen hashes of the vectors $\vec{a}$ and $\vec{b}$: $\vec{a}\vec{G}$ and $\vec{b}\vec{H}$.
* They also both have the, so-far unproven, inner product of $\vec{a}$ and $\vec{b}$: $z$.
* Note that $\vec{G}$ is the vector of bases $(G_0, G_1, \cdots, G_n)$ (that have unknown discrete logarithm).
* Note that the pedersen hash of $\vec{a}$ with base $\vec{G}$ is written as $\langle \vec{a}, \vec{G} \rangle = a_0G_0 + a_1G_1 + \cdots + a_nG_n$. You can think of that as a batching of pedersen commitments, with randomness $0$ (TODO: doesn't that mean the $a_i$ have to be uniformly random? TODO2: This is a pdersen hash, not a commitment, that we explain in another section)

## Note on notation

TODO: move to the start of this book

TODO: a symbol for hadamard product?

## The idea (preface)

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

## The idea

First, the naive way for the prover to prove that $z = \vec{a}\vec{b}$ is for the prover to send the pre-images to the pedersen hashes.
That is, to send the values in clear: $z$, $\vec{a}$, and $\vec{b}$.
The verifier can then verify that these are indeed pre-images, and then compute the inner product themselves.

The problem with this approach is that it is expensive: the vectors $\vec{a}$ and $\vec{b}$ are of size $n$, so the prover has to send $2n$ elements in $\mathbb{F}$.

We do want the prover to send pre-images of pedersen hashes, but of shorter vectors.
The idea behind the bulletproof inner argument is to repeatedly halve the sizes of the vectors $\vec{a}$ and $\vec{b}$ until they both become of size $1$.
Then the prover only has to send two pre-images of size $1$.

This will take $log_2(n)$ reductions, and each reduction will have the prover send 4 pedersen hashes.
So in total, the proof should take $4 log_2(n) + 2$ group elements?

In each reduction, the prover creates a linear combination (I think via Schwartz-Zippel?) of the vector $\vec{a}$ cut into two equal parts $\vec{a_{lo}}$ and $\vec{a_{hi}}$:

$$ \vec{a'} = \vec{a_{lo}} + \vec{a_{hi}}x $$

and with a linear combination of the pedersen basis, following the same technique:

$$ \vec{G'} = \vec{G_{lo}} $$

TODO: this is not how it works, but I'm thinking about a naive approach that doesn't work, and then an optimized approach

Notice that if we choose different linear combinations for $\vec{a'}$ and $\vec{G'}$ things combine better...

BAM

## The protocol + proofs

TODO: maybe multiplicative notation is better as we'll do this in a field? Or perhaps it'd be cool if we could switch back and forth here?

Let's take the example of vectors of size 8.
 
Split your basis $G$ into two parts: $\vec{G_{lo}} = (G_1, G_2, G_3, G_4)$ and $\vec{G_{hi}} = (G_5, G_6, G_7, G_8)$.

Same for your left vector: $\vec{a_{lo}} = (a_1, a_2, a_3, a_4)$ and $\vec{a_{hi}} = (a_5, a_6, a_7, a_8)$.

The prover sends two pedersen hash of the cross products: $A_{-1} = \vec{a_{lo}} \cdot \vec{G_{hi}}$ and $A_1 = \vec{a_{hi}} \cdot \vec{G_{lo}}$.
These are two points in the group.
For example:

$$ A_{-1} = a_1 G_5 + a_2 G_6 + a_3 G_7 + a_4 G_8 $$

From this information, both sides should be able to calculate a pedersen hash to a vector $\vec{a'} = \vec{a_{lo}} x + \vec{a_{hi}} x^2$ For some random value $x$ (given by the verifier).

We can see that this vector is half the size: 

$$ 
\vec{a'} = \begin{pmatrix*}a_1x + a_5 x^2 \\
a_2x + a_6 x^2\\
a_3x + a_7 x^2\\
a_4x + a_8 x^2
\end{pmatrix*}
$$

I think this is using the Schwartz-Zippel lemma again (instead of a random linear combination).

Then both sides can create the new basis for pedersen hashing from 

$$
\begin{align*}
\vec{G'} & = x^{-1}\vec{G_{lo}} + x^{-2}\vec{G_{hi}} \\
& = \begin{pmatrix}
  x^{-1}G_1 + x^{-2}G_5 \\ 
  x^{-1}G_2 + x^{-2}G_6 \\
  x^{-1}G_3 + x^{-2}G_7 \\
  x^{-1}G_4 + x^{-2}G_8
\end{pmatrix}
\end{align*}
$$

And both sides should be able to create the pedersen hash $A'$ of vector $\vec{a'}$ as

$$ 
\begin{align*}
A' & = x^{-1} A_{-1} + A + x A_1 = x^{-1} \vec{a_{lo}} \vec{G_{hi}} + (\vec{a_{lo}} \vec{G_{lo}} + \vec{a_{hi}} \vec{G_{hi}}) + x \vec{a_{hi}} \vec{G_{lo}} \\
& = x^{-1} (a_1 G_5 + a_2 G_6 + a_3 G_7 + a_4 G_8)
\end{align*}
$$

And indeed, the pedersen hash of $a'$ with $G'$ is the same vector:

$$ 
\begin{align*}
A' & = \vec{a'} \vec{G'} \\
& = \begin{pmatrix}
  a_1x + a_5 x^2 \\ 
  a_2x + a_6 x^2 \\
  a_3x + a_7 x^2 \\
  a_4x + a_8 x^2
\end{pmatrix} \begin{pmatrix}
  x^{-1}G_1 + x^{-2}G_5 \\ 
  x^{-1}G_2 + x^{-2}G_6 \\
  x^{-1}G_3 + x^{-2}G_7 \\
  x^{-1}G_4 + x^{-2}G_8
\end{pmatrix} \\
& \begin{split} = & (a_1x + a_5 x^2)x^{-1}G_1 + (a_1x + a_5 x^2)x^{-2}G_5 + \\ 
& (a_2x + a_6 x^2)x^{-1}G_1 + (a_2x + a_6 x^2)x^{-2}G_5 + \\
& (a_3x + a_7 x^2)x^{-1}G_1 + (a_3x + a_7 x^2)x^{-2}G_5 + \\
& (a_4x + a_8 x^2)x^{-1}G_1 + (a_4x + a_8 x^2)x^{-2}G_5 
\end{split} \\
& \begin{split} = & (a_1 + a_5 x)G_1 + (a_1 x^{-1} + a_5)G_5 + \\
& (a_1 + a_5 x)G_1 + (a_1 x^{-1} + a_5)G_5 + \\
& (a_1 + a_5 x)G_1 + (a_1 x^{-1} + a_5)G_5 + \\
& (a_1 + a_5 x)G_1 + (a_1 x^{-1} + a_5)G_5
\end{split}
\end{align*}
$$

We can do the same with $\vec{b}$, and obtain $B'$ a commitment to a vector $\vec{b'}$ half the size of $\vec{b}$:

$$ \vec{b'} = \vec{b_{lo}} x^{-1} + \vec{b_{hi}} x^{-2} $$

as well as a new base

$$ \vec{H'} = x \vec{H_{lo}} + x^2 \vec{H_{hi}} $$

The point of all these $x$s is to cancel one another at some point. Indeed:

$$ \langle \vec{a'}, \vec{b'} \rangle = \langle a_{lo} x + a_{hi} x^2, b_{lo} x^{-1} + b_{hi} x^{-2} \rangle $$

Since we are dealing with pedersen hashes, we need to compare the pedersen hashes of these with the pedersen hash of $z$, which we will have to reduce as well.

To do this, we need to send the values $z_{-1} = \langle \vec{a_{lo}} \vec{b_{hi}} \rangle$ and $z_1 = \langle \vec{a_{hi}} \vec{b_{lo}} \rangle$.

Then the verifier should be able to recompute $z'$.

At this point the prover can send the reduced pre-images $\vec{a'}$ and $\vec{b'}$ and the verifier can make sure that their inner product equals $z'$, or they can further reduce the vectors.

## Soundness of the reduction?

TKTK
