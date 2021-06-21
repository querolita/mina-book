# Compressing point equality proofs of multiple polynomials

TODO: What's a good name for this? linearization? compressions of statements? aggregating polynomial commitments? batching polynomial proofs?

Given a set of functions $f_1, \cdots, f_k$ over a field $\mathbb{F}$, we want to show that they all evaluates to $0$ for a set of points $h \in H$. In other words:

$$ \mathbb{F}, f_i(h) = 0 \; \forall i \in [[1,k]] \text{ and } \forall h \in H$$

We could send all the polynomials, and then evaluate them, but that could be a lot of polynomials to send.
Instead, we can compress that statement into showing that a single polynomial is equal to zero.

To prove that a single polynomial is equal to zero over some set, there's a technique (TODO: name? reference?): construct the target polynomial $Z_H(x) = (x - h_1) \cdot \cdots \cdot (x-h_n)$ which has the points of $H$ as roots.
Then show that our polynomial can be divided by this target polynomial.

But this does not reduce the size of our proof at all, instead we can show that on a random point.
Here's the proof that showing it on a random point suffices, that is if $\mathbb{F}$ is a large enough field.

---

Theorem:

$$
\begin{align*}
&\text{Let } f(x,y) = \sum_{i=1}^{k}x^{i-1}f_i(y).\\
&\text{If for some random } \alpha \leftarrow \mathbb{F}, \; Z_H(y) | f(\alpha, y) \; \forall y \in \mathbb{F},\\
&\text{then } \forall i \in [[1,k]] \; Z | f_i \text{ with negligible probability } k/|\mathbb{F}|.
\end{align*}
$$

---

Proof:

We can prove the contraposite:

$$
\begin{align*}
&\text{If } \exists i \text{ s.t. } Z \not{|} f_i \text{, then for a random } \alpha \leftarrow \mathbb{F},\\
& Z(y) \not{|} f(\alpha, y) \; \forall y \in \mathbb{F} \text{ with overwhelming probability } 1 - k/|\mathbb{F}|
\end{align*}
$$

So let's assume that there's an $i$ such that $Z$ does not divide $f_i$.

This implies that there's a root $y_0$ such that $Z(y_0) = 0$ but $f_i(y_0) \neq 0$.

This implies in turn that $f(x, y_0) = g(x) = \sum_{i=1}^{k}x^{i-1}f_i(y_0)$ is not the zero polynomial.

Since $g$ is of degree $k$, it has at most $k$ roots.

The probability of picking a random number $\alpha \leftarrow \mathbb{F}$ such that $g(\alpha) = 0$ is thus $k/|\mathbb{F}|$.

QED.
