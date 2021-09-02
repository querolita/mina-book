# Polynomial commitment

**THIS IS WORK-IN-PROGRESS**

Our polynomial commitment scheme is a bootleproof-type of commitment scheme, which is a mix of:

* The polynomial commitment scheme described in appendix A.1 of the [PCD paper](https://eprint.iacr.org/2020/1618).
* The zero-knowledge opening described in section 3.1 of the [HALO paper](https://eprint.iacr.org/2019/1021).

$$
\newcommand{\z}{\mathfrak{z}}
$$

## Committing to a polynomial

### Commit

- given:
    - the SRS G: a vector of `n` distinct curve points of unknown discrete logarithm
    - the SRS H: a distinc curve point of unknown discrete logarithm
    - plnm: the coefficients of a polynomial of size at most `n`
    - max: an optional upperbound we wish to enforce on the degree of the polynomial (e.g. max = 7 means the polynomial degree is 6 or less)
- do:
    - res = commit_non_hiding(plnm, max)
    - mask(res)

**Commit_non_hiding**:

- always produce a commitment `unshifted: Vec<point>`
    - split the coefficients in chunks of size at most `n` (e.g. if n is 5 and there are 11 coefficients then you should get 2 chunks of size 5 and 1 chunk of size 1)
    - for each chunk, do a multi scalar multiplication between the chunk's coefficient and the points of the SRS' G (e.g. for the coefficient vector (a, b) and the g vector `(G0, G1)` the result would be `[a]G_0 + [b]G_1`)
    - the obtained vector of curve point is the `unshifted` result
- optionally produce an upperbounded commitment `shifted: Option<point> ` if `max` is set
    - take the last chunk of the coefficient
    - if it is filled with zero, or if it of size 0 or or size n then return `None`
    - otherwise, prepend the chunk of coefficients with enough zeros to make it of size `n`, then perform a scalar multiplication to obtain the `shifted` result

**Mask**:

- given a non-hidden polynomial commitment, do:
    - for all the curve point contained in the commitment (in the vector of `unshifted`, and potentially in `shifted`) do:
        - if the curve point is zero, its associated randomness is zero (TODO: This leaks information when g is the identity! We should change this so that we still mask in this case)
        - otherwise generate a random scalar value `w`, then multiply it with the SRS' H. Then modify the commitment to now store the curve point added to wH. Store w as associated randomness.

### Polynomial evaluation (also called "opening")

Insight:

$$
\langle \vec{f} + v \cdot \vec{g}, \vec{x_1} + u \cdot \vec{x_2} \rangle = f(x_1) + v \cdot g(x_1) + u \cdot (f(x_2) + v \cdot g(x_2))
$$

Algorithm:

- args:
    - srs: an SRS
    - group_map: a map TKTK
    - plms: Vec<(DensePolynomial, Option<usize>, PolyComm)>
    - elm: Vec<F> (some elements)
    - polyscale: F (v)
    - evalscale: F (u)
    - sponge
- do:
    - if the size of the SRS' vector G is not a power of 2, append it with enough points at infinity for its size to be a power of 2
    - remember, we're creating a proof of <a, b> = z where
        - $a = \vec{f} + v \cdot \vec{g}$
        - $b = \vec{x_1} + u \cdot \vec{x_2}$
    - compute $a$ by going through each plms as `(p_i, degree_bound, omegas)` and for each non-zero `p_i` do:
        - if degree_bound is set, abort if the degree of p_i is larger than the degree_bound given. Otherwise, abort if `omegas.shifted` is not set.
        - go through the coefficients by chunk of at most the length of the SRS' G and do:
            - p: multiply all the coefficients by powers of polyscale (v) and add them up
            - blinding_factor: multiply all the omega.unshifted by powers of polyscale (v) and add them up 
            - edge-case: if degree_bound is set, and the chunk is past the degree_bound (TODO: which mean it should be the last chunk, but we don't check it):
                - pre-pend zero coefficients to the chunk so that the chunk can only use at most degree_bound Gs
                - multiply it with the next power of v and add it to p
            - $p = \sum_{i=0} v^i \cdot poly_i + v^{k+1} \cdot maybe\_shifted(poly_{k+1})$
            - $blinding\_factor = \sum_{i=0} v^i \cdot omegas.unshifted[i]$
    - compute $b = (1 + e_1 + e_1^2 + \cdots) + u(1 + e_2 + e_2^2 + \cdots) + u^2(1 + e_3 + e_3^2 + \cdots) + \cdots$
    - Create the $u$ challenge:
        - compute the inner product $\langle \vec{a}, \vec{b} \rangle$
        - absorb the `shift_scalar` of that ???
        - squeeze out `t`
        - use the group_map to transform `t` into the curve point `u`
    - make the $\vec{a}$ vector the same size as the SRS $\vec{G}$ by appending as many 0s as needed.
    - perform as many rounds as possible to obtain a 1-length vector $\vec{a'}$:
        - split G in G_lo and G_hi
        - split a in a_lo and a_hi
        - split b in b_lo and b_hi
        - generate two blinding factors rand_l and rand_r (field elements)
        - compute `l = <a_hi, G_lo> + <rand_l, H> + <a_hi, b_lo>U`
        - compute `r = <a_lo, G_hi> + <rand_r, H> + <a_lo, b_hi>U`
        - absorb l and r
        - squeeze out a challenge u, and compute its inverse (use the endo_r thing)
        - compute `a = u_inv * a_hi + a_lo`
        - compute `b = u * b_hi + b_lo`
        - compute the halved basis for the next round: `g = g_lo + u g_hi`
    - g, a, and b should all be of length 1

### Verifying a polynomial evaluation

do:
