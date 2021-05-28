# Commitments

## What's a commitment

TKTK

## Why we use commitments

In most of our zero-knowledge proofs we'll deal with commitments instead of values, because we want zero-knowledge.
These commitments are given to us via the blockchain. (TODO: examples?)

## Algebraic commitments

These commitments are also algebraic (not using a hash function like SHA-3) and as such have homomorphic properties (you can add commitments together to form another commitment of the added committed values)

## Pedersen commitments

For a value $x$, and two bases $G$ and $H$, a pedersen commitment is $xG + rH$ with randomness $r$.

It is important that you do not know the discrete logarithm of $H$ (unless $H$ produces a different subgroup?). $H$ can be obtained with a hash to curve / random oracle.

To open such a commitment, just reveal the value $x$ and $r$.

Note that the commitment relies on the discrete logarithm assumption, so in theory you might be lucky (or have a quantum computer) and figure out that $H = hG$, which would allow you to find different values $x'$ and $h'$ to open the commitment. We say that pedersen commitments are **computationally binding** and not unconditionally binding.

On the other hand, Pedersen commitments are unconditionally hiding, as there is no way even with a quantum computer to reveal what is $x$ without knowing $r$.

This is the reason why most proofs that we will see later are not proofs, but **arguments** of knowledge instead. Knowledge proofs require perfect binding.

## Multi-commitments

We can commit to several values $x_1, \cdots, x_k$ by sending the pedersen commitments of all of these values as such:

$$ x_1 G + r_1 H, \cdots, x_k G + r_k H $$

But we can instead batch/aggregate all of these commitments together as:

$$ r H + x_1 G_1 + \cdots + x_k G_k $$

with $G_1, \cdots, G_k$ independent bases with unknown discrete logarithms (can be produced).

## Pedersen hashing

Hashing needs to be deterministic, and so we can do a pedersen hashing by removing the randomness/blinding.

For one value:

$$ x G $$

For multiple values:

$$ x_1 G_1 + \cdots + x_k G_k $$

The last statement is often shortened as $\vec{x}\vec{G}$ for $\vec{x} = (x_1, \cdots, x_k)$ and $\vec{G} = (G_1, \cdots, G_k)$.

Pedersen hashing thus allow commitents that are non-hiding, but binding.
