# Permutation

The permutation argument is a mechanism to enforce the connection between the different wires of a circuit. The so-called **copy constraints**.

## Prover construction

TKTK

## Verifier construction

The verifier reconstructs the permutation argument in two parts:

* [the commitment part](#commitment-part), used to verify the evaluation sent by the prover.
* [the evaluation part](#evaluation-part), used to verify the full circuit.

### Commitment part

The verifier can reconstuct the linearized permutation commitment as:

$$
\text{Permutation} = [\text{scalar}] com(s_6)
$$

with 

$$
\begin{align}
\text{scalar} = & - z(\zeta\omega) \cdot \beta \cdot \alpha^{PERM+0} \cdot zkp(\zeta) \\
& \times (w_0(\zeta) + \gamma + \beta * s_0(\zeta)) \\
& \cdots \\
& \times (w_5(\zeta) + \gamma + \beta * s_5(\zeta)) \\
\end{align}
$$

where $s_6$ is the last permutation polynomial. Remember, there are 7 of them ($s_0$ through $s_6$).

### Evaluation part

TKTK
