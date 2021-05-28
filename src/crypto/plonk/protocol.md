# PLONK

* That's the zk-SNARK.
* There are several versions of PLONK, we use PLONK-5-wires described nowhere

## Round 1

## Round 2

## Round 3

1. challenge: $\alpha \leftarrow \mathbb{F}$ (from the verifier)
2. compute $f(x) = circuit(x) + accumulator(x) \alpha+ accumulator\_init(x) \alpha^2$ with:
    - $circuit(x) = a(x)b(x)q_M(x) + a(x)q_L(x) + b(x)q_R(x) + c(x)q_O(x) + PI(x) + q_C(x)$
    - $accumulator(x) = u1 - u2$
        + $u1(x) = (a(x) + \beta x + \gamma) (b(x)+\beta k_1 x + \gamma) (c(x)+\beta k_2 x + \gamma) z(x)$
        + $u2(x) = (a(x) + \beta S_{\sigma_1}(x) + \gamma) (b(x) + \beta S_{\sigma_2}(x) + \gamma) (c(x) + \beta S_{\sigma_3}(x) + \gamma) z(x \omega)$
    - $accumulator\_init(x) = (z(x)-1)L_1(x)$
3. compute $t(x) = f(x) / Z_H(x)$
4. commit the polynomial $t$ by commiting the evaluation of $t(x)$ at the SRS' random point $s$.

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

## Round 4

## Round 5