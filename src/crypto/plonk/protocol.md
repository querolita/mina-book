# Incremental interactive PLONK

* paper: https://eprint.iacr.org/2019/953.pdf
* commitments of polynomials is abstracted

## Overview / High-level

1. transform the problem into proving that some polynomials evaluate to zero in some domain $H$
2. commit to these polynomials
3. evaluate these polynomials to a random point $\zeta$
4. prove that the (committed) polynomials evaluate to the random points (with an opening proof)
5. optimize all of that :)

## Construction of the proof

It'd be interesting to understand how we transform a set of "requirement" iteratively from "I need to prove that I know input x in the execution of program P(x, y) = true for some public input y" to "I need to prove that these (committed) polynomials evaluate to some random point".

I wrote a simplified and incorrect sketch below, to give an idea of what I have in mind.

$$
\boxed{
    \text{I know input x in the execution of a program} \\
    \text{$P(x, y) = true$ for some public input y}
} \\
\big\downarrow \\
\boxed{
    \text{(1) I know the blank $a_i, b_i, c_i$}\\
    \text{(2) in some fixed arithmetic circuit}
} \\
\big\downarrow \\
\boxed{
    \begin{cases}
    \text{(1) I know valid $a_i, b_i, c_i$ in a constraint system of $n$ equations}\\
    \text{(2) there's a set of copy constraints (some $a, b, c$ are equal)}
    \end{cases}
} \\
\big\downarrow \\
\boxed{
    \begin{cases}
    \text{(1) I know the degree $n$ polynomials $a(x), b(x), c(x)$ that vanishes in the domain $H$ of size $n$}\\
    \text{(1) There's a constraint polynomial $r(x)$ correctly formed from these $a, b, c$ polynomials that also vanishes in the domain $H$}\\
    \text{(2) ...}
    \end{cases}
} \\
\big\downarrow \\
\boxed{
    \begin{cases}
        \text{(1) I know the degree $n$ polynomials $a(x), b(x), c(x)$} \\
        \text{(1) The polynomial $r$ created via $a, b, c$ and the public input and the selector polynomials represent our initial constraint system} \\
        \text{(1) I know a quotient polynomial $t$ s.t. $r(x)=Z_H(x)t(x)$ for each of the previous polynomials}\\
        \text{(2) ...}
    \end{cases}
}
$$

## Round 1: committing to the assignments/wires

Prover → Verifier

1. prover: generates random $b_1, b_2, b_3, b_4, b_5, b_6 \in \mathbb{F}$
1. prover: computes $a(x), b(x), c(x)$ as:
    - $a(x) = (b_1 x + b_2) \cdot Z_H + f_a(x)$
    - $b(x) = (b_3 x + b_4) \cdot Z_H + f_b(x)$
    - $c(x) = (b_5 x + b_6) \cdot Z_H + f_c(x)$
1. prover: sends $com(a), com(b), com(c)$
1. verifier: validates that these are valid commitments in $\mathbb{G}$

Explanation: 

* in this round the prover commits to their assignements polynomials (left, right, and output wires).
* $Z_H$ is the polynomial that vanishes in $H$. (It can be constructed using Lagrange interpolation.)
* Each assignment polynomials $f_i$ is blinded to add zero-knowledgness. How are the blinding values working here?
* Since the polynomial needs to remain the same on evaluations of points of $H$, the blinding also vanishes on $H$ thanks to $Z_H$.

## Round 2: committing to the permutation argument

Prover ← Verifier 
Prover → Verifier

1. verifier: sends challenge $\beta, \gamma \in \mathbb{F}$
1. prover: generates random $b_7, b_8, b_9 \in \mathbb{F}$
1. prover wants to prove that:
    * $acc_0 = 1$
    * $acc_i = acc_{i-1} (\frac{(a_i + \beta \omega^{i-1} + \gamma)(b_i + \beta k_1 \omega^{i-1} + \gamma)(c_i + \beta k_2 \omega^{i-1} + \gamma)}{(a_i + \beta \sigma_1(\omega^{i-1}) + \gamma)(b_i + \beta \sigma_2(\omega^{i-1}) + \gamma)(c_i + \beta \sigma_3(\omega^{i-1}) + \gamma)})$
1. prover: computes the permutation polynomial 
    $$ 
    \begin{align*}
    z(x) = &\; (b_7 x^2 + b_8 x + b_9) Z_H(x) + acc(x) \\
    = &\; (b_7 x^2 + b_8 x + b_9) Z_H(x) + L_1(x) + \\
    = &\; \frac{(w_1+\beta w^0+\gamma)(w_{n+1}+\beta k_1 w^0+\gamma)(w_{2n+1} + \beta k_2 w^{j-1} + \gamma)}{(w_1 + \sigma(1)\beta + \gamma)(w_{n+1}+\sigma(n+1)\beta+\gamma)(w_{2n+1}+\sigma(2n_1)\beta+\gamma)} + \cdots
    \end{align*}
    $$
1. prover: sends $com(z)$
1. verifier: validates that this is a valid commitment in $\mathbb{G}$

Explanation: 

* after committing to the assignments, we can commit to the accumulator in order to prove the copy/identity constraints.
* * $\gamma$ ensures that the two sets have the same values (not necessarily in the same order though) 
* $\beta$ ensures that the two sets have the same divisions by a permuted index (not necessarily in the same order though) 
* the combination of $\gamma$ and $\beta$ proves the permutation argument (proof in appendix A).

## Round 3: committing to the quotient polynomials

Prover ← Verifier 
Prover → Verifier

1. verifier: sends $\alpha \leftarrow \mathbb{F}$
1. prover: computes the three polynomials:
    - $circuit(x) = a(x)b(x)q_M(x) + a(x)q_L(x) + b(x)q_R(x) + c(x)q_O(x) + PI(x) + q_C(x)$
    - $accumulator\_init(x) = (z(x)-1)L_1(x)$
    - $accumulator(x) = u1(x) - u2(x)$ where
        + $u1(x) = (a(x) + \beta x + \gamma) (b(x)+\beta k_1 x + \gamma) (c(x)+\beta k_2 x + \gamma) z(x)$
        + $u2(x) = (a(x) + \beta S_{\sigma_1}(x) + \gamma) (b(x) + \beta S_{\sigma_2}(x) + \gamma) (c(x) + \beta S_{\sigma_3}(x) + \gamma) z(x \omega)$
1. prover: computes the quotient polynomials
    - $circuit\_quotient(x) = \frac{circuit(x)}{Z_H(x)}$
    - $accumulator\_init\_quotient(x) = \frac{accumulator\_init(x)}{Z_H(x)}$
    - $accumulator\_quotient(x) = \frac{accumulator(x)}{Z_H(x)}$
1. prover: since the polynomial are too large, the prover splits them into three different parts: $t_{lo}(s), t_{mid}(s), t_{hi}(s)$. We will ignore this for now.
2. prover: sends commitment to these polynomials: $com(circuit\_quotient), com(accumulator\_init\_quotient), com(accumulator\_quotient)$.
3. verifier: validates that these are valid commitments in $\mathbb{G}$

Note that in the real protocol, the prover can batch the polynomial range proof by computing $f(x) = circuit(x) + accumulator(x) \alpha+ accumulator\_init(x) \alpha^2$ with $\alpha$ a challenge from the verifier.

Explanation: 

* the first polynomial $circuit(x)$ verifies that the circuit (assignment, constant, public input and selectors) vanish in the domain
* the second polynomial $accumulator\_init(x)$ verifies that the accumulator starts at 1.
* the third polynomial $accumulator(x)$ verifies that the accumulator is well-formed
* the third polynomial also verifies that it ends up canceling out: $z(w^4) = z(1) = 1$ and $z(w^4) = z(w^3)f(x)/g(x)$

At the end, the polynomials are divided by $Z_H(x)$ to make sure that they have roots in the domain $H$. This effectively reduces the problem this way:

$$ 
\boxed{\text{we need to prove that } r(x) = 0 \; \forall x \in H} \\
\big\downarrow \\
\boxed{
    \begin{aligned}
    \text{we need to prove that } r(x) = Z_H(x)t(x) \text{ for } \\
Z_H(x) = (x-1)(x-\omega)(x-\omega^2)(x-\omega^3)\cdots \text{ and } \\
    t(x) \text{ some polynomial } 
    \end{aligned}}\\
\big\downarrow \\
\boxed{\text{we need to prove that } r(x) / Z_H(x) = t(x) \text{ is a valid polynomial}}
$$

A proof for the last statement can be formed by committing to $t$ and opening it on a random point $r$, then comparing with an opening of $f/Z_H$ at the same random point.
It is important that most of $f/Z_H$ is constructed by the verifier, restricting the freedom of the prover.
As such, PLONK has both the prover and the verifier build it together.

Note: I think in PLONK the last proof is the second step, and not the last step. In other words we prove that $r(\zeta) = Z_H(\zeta)t(\zeta)$ instead of proving that $r(\zeta)/Z_H(\zeta) = t(\zeta)$

## Round 4: evaluating the polynomial commitments at a random point $\zeta$

Prover ← Verifier 
Prover → Verifier

After committing to our polynomials, let's open it at some random challenge.

1. verifier: sends evaluation challenge $\zeta \in \mathbb{F}$
1. prover: computes and sends the opening evaluations:
    - $a(\zeta)$
    - $b(\zeta)$
    - $c(\zeta)$
    - $S_{\sigma_1}(\zeta)$
    - $S_{\sigma_2}(\zeta)$
    - $circuit\_quotient(\zeta)$
    - $accumulator\_init\_quotient(\zeta)$
    - $accumulator\_quotient(\zeta)$
    - $z(\zeta \cdot \omega)$
1. verifier: validates that these are correct evaluations in $\mathbb{F}$

TODO: Note about the lineraisation polynomial $r(x)$ in the real PLONK protocol.

Explanation:

* The prover now opens the commitments at a random point.
* TODO: $z(\zeta \cdot \omega)$ what is this?
- TODO: Both the prover and the verifier should be able to compute $S_{\sigma_1}(\zeta)$, $S_{\sigma_2}(\zeta)$ right?



## Round 5: Opening proofs

Prover ← Verifier 
Prover → Verifier

Note that this round is actually multiple subrounds in our version of PLONK, as we use the inner product argument for polynomial commitments and opening proofs. An opening proof in an inner product argument takes $log(n)$ moves for $n$ the degree of a polynomial.

1. verifier: sends opening challenge $v \in \mathbb{F}$
1. prover: computes and sends the opening proofs for:
    - $a$
    - $b$
    - $c$
    - $S_{\sigma_1}$ -> why? isn't this public?
    - $S_{\sigma_2}$ -> optimization perhaps?
    - $circuit\_quotient$
    - $accumulator\_init\_quotient$
    - $accumulator\_quotient$

--- 

3. verifier: evaluates target/vanishing polynomial $Z_H(\zeta) = \zeta^n - 1$
4. verifier: evaluates lagrange polynomial $L_1(\zeta) = \frac{\zeta^n-1}{n(\zeta-1)}$
5. verifier: evaluates public input polynomial $PI(\zeta) = \sum_{i \in l} w_i L_i(\zeta)$
6. verifier: evaluates the quotient polynomial
    $$
    x
    $$

Explanation:

* In the final round, the exchange creates a proof that the committed polynomials do evaluate to the previous round evaluations


## Bonus: the PLONK paper (interactive version)

![](https://i.imgur.com/km2h4yZ.png)
