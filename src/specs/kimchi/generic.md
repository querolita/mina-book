# Generic

The generic gate is the vanilla PLONK gate, which allows us to implement a number of different gates depending on how it's called:

* addition gate: the first three columns are used to represent the left, right, and output wire
* multiplication gate: TKTK
* constant gate: only the most-left column is set to what the constant value should be
* public input gate: only the most-left column is set

## Prover construction

TKTK

## Verifier construction

The verifier reconstructs the generic gate as:

$$
\begin{align}
\text{Generic} &= [w_0(\zeta)w_1(\zeta)]com(qm) \\
& + [w_0(\zeta)]com(qw_0) + [w_1(\zeta)]com(qw_1) + [w_2(\zeta)]com(qw_2) \\
& + [1]com(qc)
\end{align}
$$

where the first line is the multiplication, the second line is the addition and the output, and the third line is the constant.

