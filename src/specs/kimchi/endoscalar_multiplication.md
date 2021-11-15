# Endoscalar multiplication

**THIS IS WORK-IN-PROGRESS**

Since the dominating cost of proof verification is multiplication of group elements by challenges (i.e. scaling), Mina employ endoscalar multiplication (a.k.a. endoscaling) as an optimization.  Specifically, inside the circuit scaling by a 128-bit value costs about 50 rows whilst endoscaling by a 128-bit value requires only 32 rows-- a 34% savings.

## How

Define an endomorphism
* $\phi(P) = [\zeta_q]P$ for $\zeta_q \in \mathbb{F}_q$
* $\phi(Q) = [\zeta_p]Q$ for $\zeta_p \in \mathbb{F}_p$ 

where $\zeta_p$ and $\zeta_q$ are of multiplicative order 3.

For a point $P = (x, y) \in \mathbb{F}_p$, we have $\phi(P) = (\zeta_q x, y)$

* Given verifier challenge $r \in \{0, 1\}^{\lambda}$
* Rather than perform scalar multiplication of $r$ and $P \in E \setminus \{ \mathcal{O} \}$
* Use $\phi$ to multiply $P$ by another scalar dependent on $r$ like so...

## Algorithm

```rust
fn endo_mul(r, P) {
    let mut acc = (\phi(P) + P).double(); 
    for i from \lambda/2 - 1 to 0 {

        let S = match r[2i] {
            0 => P.negate(),
            _ => P,
        }

        if r[2i + 1] != 0 {
            S = \phi(S)
        }

        acc += acc + S;
    }
    return acc;
}
```

**TODO** Update against [this](https://github.com/o1-labs/proof-systems/blob/a41cc9fa0fe8d269c03616491f02e2fc6159ac46/oracle/src/sponge.rs#L31)

# References

1. [Recursive Proof Composition without a Trusted
Setup](https://eprint.iacr.org/2019/1021.pdf)