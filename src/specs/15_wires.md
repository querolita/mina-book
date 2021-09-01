# 15-wires



## Proof Creation

Idea: it's fine if something crash here, not in verifier tho

Given:

* **`witness`**: a witness, which is a list of columns (15) containing n group elements
* **`group_map`**: a group map, an operation TKTK
* **`index`**: an index, information related to the circuit
* **`prev_challenges`**: ???

1. **Create the public polynomial** (via interpolation) as the polynomial that evaluates to the public values for x = domain[0..public.len()-1]. The public values are taken from the witness' first column. Then negate the public polynomial (really?)
2. **Compute the witness polynomials**. Interpolate each column of the witness into a polynomial that evaluates to the entry's value when given points of the domain d1. The first column polynomial will also containt the public values (weird no?)
3. **Commit the wire values**. For each witness polynomial, commit them via the following process:
    a. perform a multi scalar multiplication (MSM) with the coefficient of the polynomial and the SRS's base (G_1, G2, ..., G_N) to produce a non-hidding commitment
    b. generate a random group element w
    c. add [w]H to the non-hidding commitment to produce a hiding commitment.
4. **Absorb transcript and produce $\beta$, $\gamma$**.
    - Absorb a non-hiding commitment of the public input (absorb x coordinate, then y coordinate, as two field elements)
    - Absorb each witness polynomial commitment in the same way
    - Squeeze out two field elements: beta and gamma (the way squeeze is implemented involves sponge_5wires, look into that)
5. **Compute permutation aggregation polynomial**.
    - perm_aggreg() <- how does this work?
    - compute the commitment of the permutation polynomial `z`
6. **Absorb the z commitment into the argument and query $\alpha$**
    - absorb the z polynomial commitment
    - produce the alpha challenge (a group element)
    - use ScalarChallenge::to_field with the SRS' endo_r to create alpha'
    - compute powers of alpha' (from alpha'^0 to alpha'^59)
7. **Evaluate polynomials over domains**
    - ???
8. **Compute quotient polynomial**.
    - Compute the quotient polynomial of all gates and the permutation polynomial individually. These will sometimes use other domains! Why? I guess it requires more than n evaluations?
        a. Permutation
        b. Generic
        c. Poseidon
        d. EC addition
        e. EC doubling
        f. endoscaling
        g. scalar multiplication
    - collect contribution evaluations
        - t4 = add + mul4 + pos4 + gen + doub4
        - t8 = perm + mul + emul8 + pos8 + doub8
    - divide contributions with vanishing polynomial
        - f = t4.interpolate() + t8.interpolate() + genp + posp
        - t = f/Zh
        - t += bnd
9. Commit the quotient polynomial
    - perform an upperbounded polynomial commitment with upperbound = max_quot_size (which is hardcoded in the index/circuit)
10. Absorb and produce zeta
    - absorb commitment of t (unshifted)
    - absorb a number of dummy group elements = (0, 0), exactly `max_t_size - t_comm.0.unshifted.len() of them`. Why? This looks like padding.
    - if t_comm.shifted is zero, then absorb dummy again, otherwise absorb t_comm.shifted
    - squeeze to obtain zeta challenge
    - use ScalarChallenge::to_field with endo_r to obtain zeta' (why? we don't always scalar mul with zeta no?)
11. Evaluate the committed polynomials (witness and quotient) at zeta
    - compute zeta * omega (where omega is the generator of the domain d1)
12. Compute and evaluate the linearization polynomial, (what's that again?)
13. Query opening scalar challenges
    - squeeze v and create v' with the endo technique
    - squeeze u and create u' with the endo technique
14. Compute the opening proof (ok that's where we aggregate all the openings and what's to prove)
    - prev_challenges { chals, comm } -> (???, comm.unshifted.len())
    - non_hiding = commitment of zero?

## Proof Verification

- if the length of the proofs is 0, return true
- create params from proofs, for each proof as (index, lgr_comm, proof) do:
    - compute public input commitment as `- proof.public[1] * L_1(x) - proof.public[2] * L_2(x) - ...`
    - create a sponge and create all the challenges from it:
        - oracles.beta/gamma: absorb the public and the witness commitments, then squeeze out oracles.beta and oracles.gamma
        - oracles.alpha: absorb the z commitment (permutation commitment) and squeeze alpha_chal. Derive oracles.alpha from it (using endo_r)
        - oracles.zeta: absorb padded t commitment, and squeeze zeta_chal. Derive oracles.zeta from it (using endo_r)
        - fq_sponge: the state of the Fq sponge BEFORE the last evaluation
        - digest: the squeeze of the Fq sponge AFTER the last evaluation
        - create an fr_sponge and absorb the digest
        - alpha: pre-compute the powers of oracles.alpha starting from alpha^2
        - retrieve all the domain index for public
        - p_eval: 
            ```
            [
                [
                (zeta^n - 1) * 
                [-pub[0]/(n(zeta-1)) - pub[1]/(n(zeta - w)) - pub[2]/(n(zeta - w^2)) - ...] 
                // are these really the lagrange polynomials? (zeta^n-1)/(n(zeta-X))
                ],
                [
                    // same, but with zeta * w
                ]
            ]
            ```
        - absorb the (p_eval[i], proof.evals[i]) and squeeze out v_chal and u_chal. Derive oracles.v and oracles.u from them (using endo_r)
        - ...
        - evlp: [zeta^n, (zeta*w)^n] where n is the max_poly_size
        - polys:
        - zeta1:
        - combined_inner_product:
    - evals = take the proof evaluations of segments of l, f, etc. (proofs.evals[i]) and combine them with powers of z^n or (zw)^n (evlp[i]) // this will help linearize the rest
    - compute the lineraization polynomial commitments required to check the opening proofs:
        - permutation
        - generic
        - poseidon
        - EC addition
        - EC doubling
        - endoscalar multiplication
        - scalar multiplication
        - f
    - check the linearization polynomial consistency (probably $f(z) = t(z)Z_H(z)$)
        - $public(\z) = 0$ or the evaluation of the public polynomial at $\z$ if there's a public input
        - $missing\_perm = (w[6](\z) + \gamma) \cdot z(\omega\z) \cdot \alpha^{PERM0} \cdot zkpm(\z) \cdot \sum_{i=0}^5 (\beta \cdot s[i](\z))  + witness[i] + \gamma$
            // only involves the witness that are part of the permutation argument (7 of them)
        - $missing\_perm2 = \alpha^{PERM0} \cdot zkpm(\z) \cdot z(\z) \cdot \sum_{i=0}^6 (\gamma + \beta \cdot \z \cdot shift[i] + w[i])$ // shift is for the cosets
        - $left = [f(\z) + public(\z) - missing\_perm + missing\_perm2 - t(\z) \cdot (\z^n - 1)](\z - \omega^{n-3})(\z - 1)$
        - $right = [(\z^n - 1) \cdot \alpha^{PERM1} \cdot (\z - \omega^{n-3}) + (\z^n - 1) \cdot \alpha^{PERM2} \cdot (\z - 1)] (1 - z(\z))$
        
        - ensure $left = right$
    - check the aggregated opening proof
    
    [f(zeta) + pub(zeta) + permutation_stuff - t(zeta) * (zeta^n - 1)](zeta - w^{n-3})(zeta - 1)
    
    compare that with PLONK-3-wires:
    
    ![](https://i.imgur.com/NolFVxl.png)
