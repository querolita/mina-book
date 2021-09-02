# 15-wires

**THIS IS WORK-IN-PROGRESS**

This document specifies the 15-wires variant of PLONK.

## Overview

The document follows the following structure:

1. **Setup**. A one-time setup for the proof system.
2. **Per-circuit setup**. A one-time setup for each circuit that are used in the proof system.
3. **Proof creation**. How to create a proof.
4. **Proof verification**. How to verify a proof.

## Dependencies

### Polynomial Commitments

Refer to the [specification on polynomial commitments](). We make use of the following functions from that specification:

- `PolyCom.non_hiding_commit(poly) -> PolyCom::NonHidingCommitment`
- `PolyCom.commit(poly) -> PolyCom::HidingCommitment`
- `PolyCom.evaluation_proof(poly, commitment, point) -> EvaluationProof`
- `PolyCom.verify(commitment, point, evaluation, evaluation_proof) -> bool`

### Poseidon hash function

Refer to the [specification on Poseidon](). We make use of the following functions from that specification:

- `Poseidon.init(params)`
- `Poseidon.update(field_elem)`
- `Poseidon.finalize() -> FieldElem`

## Setup

This section introduces the global setup which is used for any circuit. For each circuit that a prover and a verifier wants to use the protocol for, a per-circuit setup will have to be performed as well. See the [next section on per-circuit setup](#per-circuit-setup).

### Data Structures

```rust
struct SRS<G: CommitmentCurve> {
    /// Curve points for committing polynomials
    g: Vec<G>, 
    /// Curve point for blinding
    h: G,      
    // Coefficients for the curve endomorphism
    endo_r: G::ScalarField,
    endo_q: G::BaseField,
}
```

## Per-circuit setup

### Data Structures

```rust
struct ConstraintSystem<F: FftField> {
    // Basics
    // ------

    /// number of public inputs
    public: usize,
    /// evaluation domains
    domain: EvaluationDomains<F>,
    /// circuit gates
    gates: Vec<CircuitGate<F>>,

    // Polynomials over the monomial base
    // ----------------------------------

    /// permutation polynomial array
    sigmam: [DP<F>; PERMUTS],
    /// zero-knowledge polynomial
    zkpm: DP<F>,

    // Generic constraint selector polynomials
    // ---------------------------------------

    /// linear wire constraint polynomial
    qwm: [DP<F>; GENERICS],
    /// multiplication polynomial
    qmm: DP<F>,
    /// constant wire polynomial
    qc: DP<F>,

    // Poseidon selector polynomials
    // -----------------------------

    /// round constant polynomials
    rcm: [[DP<F>; SPONGE_WIDTH]; ROUNDS_PER_ROW],
    /// poseidon constraint selector polynomial
    psm: DP<F>,

    // ECC arithmetic selector polynomials
    // -----------------------------------

    /// EC point addition constraint selector polynomial
    addm: DP<F>,
    /// EC point doubling constraint selector polynomial
    doublem: DP<F>,
    /// mulm constraint selector polynomial
    mulm: DP<F>,
    /// emulm constraint selector polynomial
    emulm: DP<F>,

    //
    // Polynomials over lagrange base
    //

    // Generic constraint selector polynomials
    // ---------------------------------------

    /// left input wire polynomial over domain.d4
    qwl: [E<F, D<F>>; GENERICS],
    /// multiplication evaluations over domain.d4
    qml: E<F, D<F>>,

    // permutation polynomials
    // -----------------------

    /// permutation polynomial array evaluations over domain d1
    sigmal1: [Vec<F>; PERMUTS],
    /// permutation polynomial array evaluations over domain d8
    sigmal8: [E<F, D<F>>; PERMUTS],
    /// SID polynomial
    sid: Vec<F>,

    // Poseidon selector polynomials
    // -----------------------------

    /// poseidon selector over domain.d4
    ps4: E<F, D<F>>,
    /// poseidon selector over domain.d8
    ps8: E<F, D<F>>,

    // ECC arithmetic selector polynomials
    // -----------------------------------

    /// EC point addition selector evaluations w over domain.d4
    addl: E<F, D<F>>,
    /// EC point doubling selector evaluations w over domain.d8
    doubl8: E<F, D<F>>,
    /// EC point doubling selector evaluations w over domain.d4
    doubl4: E<F, D<F>>,
    /// scalar multiplication selector evaluations over domain.d4
    mull4: E<F, D<F>>,
    /// scalar multiplication selector evaluations over domain.d8
    mull8: E<F, D<F>>,
    /// endoscalar multiplication selector evaluations over domain.d8
    emull: E<F, D<F>>,

    // Constant polynomials
    // --------------------
    
    /// 1-st Lagrange evaluated over domain.d8
    l1: E<F, D<F>>,
    /// 0-th Lagrange evaluated over domain.d4
    // TODO(mimoo): be consistent with the paper/spec, call it L1 here or call it L0 there
    l04: E<F, D<F>>,
    /// 0-th Lagrange evaluated over domain.d8
    l08: E<F, D<F>>,
    /// zero evaluated over domain.d8
    zero4: E<F, D<F>>,
    /// zero evaluated over domain.d8
    zero8: E<F, D<F>>,
    /// zero-knowledge polynomial over domain.d8
    zkpl: E<F, D<F>>,

    /// wire coordinate shifts
    shift: [F; PERMUTS],
    /// coefficient for the group endomorphism
    endo: F,

    /// random oracle argument parameters
    fr_sponge_params: ArithmeticSpongeParams<F>,
}
```

## Proof Creation

### Data structure

```rust
struct ProverProof<G: AffineCurve> {
    /// polynomial commitments
    commitments: ProverCommitments<G>,
    /// batched commitment opening proof
    proof: OpeningProof<G>,
    /// chunked polynomial evaluations (TODO: specify)
    evals: [ProofEvaluations<Vec<Fr<G>>>; 2],
    /// public part of the witness
    public: Vec<Fr<G>>,
    /// The challenges underlying the optional polynomials folded into the proof
    prev_challenges: Vec<(Vec<Fr<G>>, PolyComm<G>)>,
}
```

### Protocol

Idea: it's fine if something crash here, not in verifier tho

Given:

* **`witness`**: a list of 15 vectors, each containg $n$ group elements. Note that the first `cs.public` elements of the first vector contain the public inputs (and so the first `cs.public` elements of the other vectors must be set to 0).
* **`group_map`**: (TODO: move this to setup?)
* **`prover_index`**: a structure containing:
  - constraint system: the constraint system of a specific circuit
  - SRS: (TODO: this should move out of the index)
  - max_poly_size: (TODO: this should be in SRS)
  - max_quot_size: (TODO: I think we don't need this anymore after maller's optimization)
  - fq_sponge_params: the sponge parameters to use for fiat-shamir. Fq indicates the base field, as we are mostly absorbing commitments, which are points on the curve.
* **`prev_challenges`**: related to recursion (TODO: make this an Option/type/rename?)

1. **Create the public polynomial** as the polynomial that, when evaluated at the first `cs.public` elements of the domain `cs.domain`, evaluates to the first `cs.public` elements of the witness' first column. Then negate that polynomial (TODO: we can simplify this by subtracting it later.)
2. **Compute and commit to the 15 witness polynomials**.
    - Interpolate each column of the witness into a polynomial that evaluates to the entry's value when given points of the `cs.domain`.
    - Produce commits for each of the witness polynomials using `PolyCom.commit()`.
3. **Absorb the public input and the witness and produce $\beta$ and $\gamma$**.
    - Create a non-hiding commitment of the public input via `PolyCom.non_hiding_commit()`.
    - Absorb it (absorb the coordinates as two field elements) via `Poseidon.update()`. Assumption here is that the commitment is always the same size (TODO: verify/enforce this)
    - Absorb each witness polynomial commitment in the same way.
    - Squeeze two field elements: $\beta$ and $\gamma$, each of size 128 bits using (TODO: specify how to do that)
4. **Compute and commit to the permutation polynomial**.
    - Compute the permutation polynomial `z` (TODO: specify)
    - Compute a commitment of the permutation polynomial `z` using `PolyCom.commit()`
5. **Absorb the permutation and produce $\alpha$**
    - Absorb the z polynomial commitment. Assumption here is that the commitment to z is always the same size (TODO: verify/enforce this) 
    - Squeeze out a field element `alpha_challenge` of size 128 bits (TODO: specify)
    - Use the endomorphism to produce $\alpha$ (TODO: specify)
6. **Produce the gate polynomials for the circuit**.
    a. Permutation
    b. Generic
    c. Poseidon
    d. EC addition
    e. EC doubling
    f. Endoscaling
    g. Scalar multiplication
7. **Compute and commit to the quotient polynomial $t$**.
    - Add all of the polynomials together then divide the result with the vanishing polynomial (TODO: specify). 
    - Ensure that the division doesn't have a non-zero remainder.
    - Perform an upperbounded polynomial commitment with `upperbound = max_quot_size` (which is hardcoded in the index/circuit) via `PolyCom.commit()`.
8.  **Absorb the quotient polynomial and produce $\zeta$**
    - Absorb the commitment of $t$ (TODO: specify how to do that when a commitment is several points)
    - Absorb "fake" group elements $(0, 0)$ as padding to make sure that exactly `max_t_size` curve points are absorbed with the commitment of $t$. This is to make sure that we always absorb the same number of curve points no matter the length of the commitment of $t$.
    - If the shifted value of the $t$ commitment is the point-at-infinity, absorb the "fake" group element $(0,0)$. Otherwise, absorb the shifted value.
    - Squeeze to obtain the zeta challenge.
    - Use the endomorphism to produce $\zeta$ from the zeta challenge.
9. **Evaluate the committed polynomials at $\zeta$ and $\zeta \omega$ (TODO: specify what $\omega$ is).** For each polynomials produce two types of evaluations: normal evaluations (to use for the linearization step) and chunked evaluations to use for the proof. (TODO: specify chunked evaluations)
    - The 15 witness polynomials $w_0, \cdots, w_14$
    - The permutation polynomial $z$
    - The quotient polynomial $t$
10. **Evaluate the $\sigma$ polynomials at $\zeta$ and $\zeta \omega$ as well**. This is to help the verifier.
11. **Produce the linearized gate polynomials for the circuit**.
    a. Permutation
    b. Generic
    c. Poseidon
    d. EC addition
    e. EC doubling
    f. Endoscaling
    g. Scalar multiplication
12. **Produce the linearized and chunked evaluations of $f$**:
    - Add all the linearized gate polynomials together to produce $f$.
    - Evaluate (in chunks) $f$ at $\zeta$ and $\zeta \omega$.
13. **Switch sponge**. (TODO: why do we switch sponge?)
    - Save the state of the sponge as `fq_sponge_before_evaluations` for the proof.
    - Create an `FrSponge` with `cs.fr_sponge_params`
    - Squeeze the fq_sponge, convert the field element to Fr, and absorb that field element with the fr_sponge (TODO: specify the conversion)
14. **Evaluate the public input at $\zeta$ and $\zeta \omega$**.
16. **Absorb the public input and the chunked evaluations and produce the challenges $v$ and $u$ using the Fr Sponge**
    - absorb the evaluations at $\zeta$ of:
      - public input $p(\zeta)$ (TODO: why do we re-absorb the public input?)
      - $w_0(\zeta)$
      - $w_1(\zeta)$
      - $w_2(\zeta)$
      - $w_3(\zeta)$
      - $w_4(\zeta)$ (TODO: what about the rest? This is for 5-wires)
      - $z(\zeta)$
      - $t(\zeta)$
      - $f(\zeta)$
      - $s_0(\zeta)$
      - $s_1(\zeta)$
      - $s_2(\zeta)$
      - $s_3(\zeta)$ (TODO: what about the rest? This is for 5-wires)
      - (TODO: what about s_4?)
    - absorb the evaluations at $\zeta \omega$ of:
      - public input $p(\zeta \omega)$
      - $w_0(\zeta \omega)$
      - $w_1(\zeta \omega)$
      - $w_2(\zeta \omega)$
      - $w_3(\zeta \omega)$
      - $w_4(\zeta \omega)$ (TODO: what about the rest? This is for 5-wires)
      - $z(\zeta \omega)$
      - $t(\zeta \omega)$
      - $f(\zeta \omega)$
      - $s_0(\zeta \omega)$
      - $s_1(\zeta \omega)$
      - $s_2(\zeta \omega)$
      - $s_3(\zeta \omega)$ (TODO: what about the rest? This is for 5-wires)
      - (TODO: what about s_4?)
    - squeeze the v challenge and create $v$ from it using the endomorphism (TODO: specify)
    - squeeze the u challenge and create $u$ from it using the endomorphism (TODO: specify)
17. Compute the aggregated evaluation proof for evaluation points $\zeta$ and $\zeta \omega$ of the following polynomials (TODO: specify how). Use $v$ and $u$ as TKTK.
    - the prev_challenges (TODO: specify)
    - the public polynomial $p$
    - the 15 witness polynomials $w_0, \cdots, w_14$
    - the permutation poylnomial $z$
    - the quotient polynomial $t$
18. Set the final proof to: (TODO: shouldn't it contain a hash of the constraint system/circuit as an identifier as well? there could be attacks when you try to verify a wrong proof with a malicious or wrong constraint system?)
    - commitments are the commitments to:
      - the 15 witness polynomials $w_0, \cdots, w_14$
      - the permutation polynomial $z$
      - the quotient polynomial $t$
    - the evaluation proof
    - the chunked evaluations of
      - s
      - w
      - z
      - t
      - f
    - the public inputs
    - prev_challenges (TODO: specify)

## Proof Verification

Given:

* group_map
* a vector of proofs:
  * a number of commitments
  * a proof
  * and a verifier index containing:
    * domain
    * max_poly_size
    * max_quote_size
    * SRS
    * pre-computed commitments:
      * sigma_comm
      * qw_comm
      * qm_comm
      * qc_comm
      * rcm_comm
      * psm_comm
      * add_comm
      * double_comm
      * mul_comm
      * emul_comm
    * shifts
    * zero-knowledge polynomial
    * w
    * endo
    * fr_sponge_params
    * fq_sponge_params


do:

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
        - $public(\zeta) = 0$ or the evaluation of the public polynomial at $\zeta$ if there's a public input
        - $missing\_perm = (w[6](\zeta) + \gamma) \cdot z(\omega\zeta) \cdot \alpha^{PERM0} \cdot zkpm(\zeta) \cdot \sum_{i=0}^5 (\beta \cdot s[i](\zeta))  + witness[i] + \gamma$
            // only involves the witness that are part of the permutation argument (7 of them)
        - $missing\_perm2 = \alpha^{PERM0} \cdot zkpm(\zeta) \cdot z(\zeta) \cdot \sum_{i=0}^6 (\gamma + \beta \cdot \zeta \cdot shift[i] + w[i])$ // shift is for the cosets
        - $left = [f(\zeta) + public(\zeta) - missing\_perm + missing\_perm2 - t(\zeta) \cdot (\zeta^n - 1)](\zeta - \omega^{n-3})(\zeta - 1)$
        - $right = [(\zeta^n - 1) \cdot \alpha^{PERM1} \cdot (\zeta - \omega^{n-3}) + (\zeta^n - 1) \cdot \alpha^{PERM2} \cdot (\zeta - 1)] (1 - z(\zeta))$
        
        - ensure $left = right$
    - check the aggregated opening proof
    
    [f(zeta) + pub(zeta) + permutation_stuff - t(zeta) * (zeta^n - 1)](zeta - w^{n-3})(zeta - 1)
    
    compare that with PLONK-3-wires:
    
    ![](https://i.imgur.com/NolFVxl.png)
