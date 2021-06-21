# PLONK

* That's the zk-SNARK.
* There are several versions of PLONK, we use PLONK-5-wires described nowhere

## Prover
### Round 1

* generate random $b_1, b_2, b_3, b_4, b_5, b_6 \in \mathbb{F}$
* compute $a(x), b(x), c(x)$ as:
    - $a(x) = (b_1 x + b_2) \cdot Z_H + f_a(x)$
    - $b(x) = (b_3 x + b_4) \cdot Z_H + f_b(x)$
    - $c(x) = (b_5 x + b_6) \cdot Z_H + f_c(x)$
* output $[a(s)], [b(s)], [c(s)]$

Explanation: this commits the assignement polynomials. This is the witness I guess.

### Round 2

* generate random $b_7, b_8, b_9 \in \mathbb{F}$
* get challenge $\beta, \gamma \in \mathbb{F}$
* compute $z(x) = (b_7 x^2 + b_8 x + b_9) Z_H(x) + acc(x)$
* output $[z(s)]$

After committing to the assignment, we can commit to the accumulator.

### Round 3

1. challenge: $\alpha \leftarrow \mathbb{F}$ (from the verifier)
2. compute $f(x) = circuit(x) + accumulator(x) \alpha+ accumulator\_init(x) \alpha^2$ with:
    - $circuit(x) = a(x)b(x)q_M(x) + a(x)q_L(x) + b(x)q_R(x) + c(x)q_O(x) + PI(x) + q_C(x)$
    - $accumulator(x) = u1 - u2$
        + $u1(x) = (a(x) + \beta x + \gamma) (b(x)+\beta k_1 x + \gamma) (c(x)+\beta k_2 x + \gamma) z(x)$
        + $u2(x) = (a(x) + \beta S_{\sigma_1}(x) + \gamma) (b(x) + \beta S_{\sigma_2}(x) + \gamma) (c(x) + \beta S_{\sigma_3}(x) + \gamma) z(x \omega)$
    - $accumulator\_init(x) = (z(x)-1)L_1(x)$
3. compute $t(x) = f(x) / Z_H(x)$
4. commit the polynomial $t$ by commiting the evaluation of $t(x)$ at the SRS' random point $s$. Since the polynomial is too large, we split it into three different parts: $t_{lo}(s), t_{mid}(s), t_{hi}(s)$.

Note that: 

* the first polynomial $circuit(x)$ verifies that the circuit (assignment, constant, public input and selectors) vanish in the domain
* the second polynomial $accumulator(x)$ verifies that the accumulator is well-formed AND that it ends up canceling out ($z(1) = 1 = z(w^4) = z(w^3)f(x)/g(x)$)
* the third polynomial $accumulator\_init(x)$ verifies that the accumulator starts at 1.

then, they are all divided by $Z_H(x)$ to make sure that they have roots in the domain $H$. This effectively reduces the problem this way:

we need to prove that $f(x) = 0 \; \forall x \in H$

to

we need to prove that $f(x) = Z_h(x)t(h)$ for $Z_h(x) = (x-1)(x-\omega)(x-\omega^2)(x-\omega^3)$ and $t(h)$ some polynomial

to

we need to prove that $f(x) / Z_h(x) = t(x)$ is a valid polynomial (if you can evaluate it at some point, and the result is an integer, that's a win).

the three polynomials are also combined into a single polynomial $f(x)$ using something that has no name I believe, but uses the Schwartz-Zippel lemma. Basically, we create the polynomial $f(x,y) = f_1(x) + f_2(x)y + f_3(x)y^2$ then evaluates it at a random point $y$ and show that it is still $0$.

### Round 4

After committing to our polynomials, let's open it at some random challenge.

* compute evaluation challenge $\zeta \in \mathbb{F}$
* compute opening evaluations:
    - $\bar{a} = a(\zeta)$
    - $\bar{b} = b(\zeta)$
    - $\bar{c} = c(\zeta)$
    - $\bar{S_{\sigma_1}} = S_{\sigma_1}(\zeta)$
    - $\bar{S_{\sigma_2}} = S_{\sigma_2}(\zeta)$
    - $\bar{t} = t(\zeta)$
    - $\bar{z_{\omega}} = z(\zeta \cdot \omega)$
* compute linearization polynomial $r(x) = ...$
* compute linearization evaluation $\bar{r} = r(\zeta)$
* output $(\bar{a},\bar{b},\bar{c},\bar{S_{\sigma_1}},\bar{S_{\sigma_2}},\bar{t},\bar{z_{\omega}},\bar{r})$

### Round 5

## Verifier

1. validate that everything receives is part of our group (normal point validation)
2. validate that the received evaluations are in our field, but why check about the permutations also? Isn't that public? Maybe only the prover has access to the random evaluation point?
3. what's this? the public input I think? 
4. compute the zero-polynomial evaluation at zeta, so we know zeta!
5. compute lagrange poly at zeta, sure
6. public input is just a poly that takes the value of the different inputs at different points (built via lagrange poly)
7. quotient polynomial (everything divided by target polynomial), that proves that indeed that poly vanishes in H. What's in there exactly? The whole thing no? 
    - It's the evaluation of r at zeta, and r is indeed the whole thing (so the result, we still need to make sure it's the same as the individual commitments)
8. compute the evaluation of our big/batched polynomial at the zeta point using our commitments to the selector poly, and the received commitments to a, b, c, permutations, and also the v_i
9.
10.
