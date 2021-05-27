# Compressing point equality proofs of multiple polynomials

**Re-write this, as it should work over a set of points $H$**.

Given a set of functions $f_1, \cdots, f_k$ over a field $\mathbb{F}$, we want to show that they all evaluates to $0$ for some point $h$. In other words:

$$ \text{for some } h \in \mathbb{F}, f_i(h) = 0 \; \forall i \in [[1,k]] $$

We could send all the polynomials, and then evaluate them, but that could be a lot of polynomials to send.
Instead, we can compress that statement into showing that a single polynomial is equal to zero.
(TODO: To prove that a polynomial is equal to zero, there's a different technique.)

The reduction is to show that the following polynomial $\tilde{f}$ is equal to $0$ at some random point $\alpha \leftarrow \mathbb{F}$:

$$ \tilde{f}(x) = f_1(h) + f_2(h) x + f_3(h) x^2 + \cdots + f_k(h) x^{k-1} $$

Claim: the probability that $\tilde{f} = 0$ while there's some $f_i(h) \neq 0$ is negligible (if $\mathbb{F}$ is large and \tilde{f} is correctly constructed, that is).

**Proof**:

1. Assume that for some random $\alpha \in \mathbb{F}$ we have $\tilde{f}(\alpha) = 0$. We want to show that the probability that $P(\tilde{f}(\alpha) = 0 \text{ and } \exists i \text{ s.t. } f_i(h) \neq 0)$ is negligible.
1. Assume that $\exists i$ such that $f_i(h) \neq 0$.
1. From the previous step, $\tilde{f}(x)$ is a non-zero polynomial that evaluates to $0$ for some random point $\alpha$.
1. The probability that $\tilde{f}$ is not the zero-polynomial and that it evaluates to $0$ for some random point $\alpha$ is $deg(\tilde{f}) / |\mathbb{F}|$.
    1. TODO: Schwartz–Zippel lemma
1. So $\tilde{f}$ is the zero polynomial with overwhelming probability.
