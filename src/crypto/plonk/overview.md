# Overview

- To prove things in Mina we use PLONK, or at least a variant of PLONK.
- In this section we explain the variant only (give it a name! mlonk?)
- It is not a zk-SNARK as it is not succint in the proof size (11kb instead of 1kb?)
- Essentially what PLONK allows you to do is to take a program (with inputs and outputs) and take a snapshot of its execution. Then, you can remove parts of the inputs, but prove to someone that the execution is correct for the remaining inputs and outputs of the snapshot.
- There are a lot of steps involved in the transformation of "I know an input to this program such that when executed with these other inputs it gives this output" to "here's a sequence of operations you can execute to convince you of that".
- The rest of this section will go through the different steps of that ^ + the optimizations + everything else

we can use the following terms:

- succint: short proof size (log n for a circuit of n gates)
- fast: verifier is fast to verify (log)

note that the prover can't be faster than linear, as they at least have to read the circuit.

in the rest:

- we will define every tool needed
- the whole point is to go from program -> arithmetic circuit -> constraints -> polynomials, as polynomials are what we can prove later
- then we will explain how polynomial commitments work and how we can prove things succintly (to some extent) with them.
- everything will be used in the final protocol section that describes our variant of PLONK.

TODO: what are the new parts in our variants? What are the removed parts?

## Stuff that needs to be written somewhere

* you choose the domain size when creating the constraint system for a circuit. Thus, the domain size depends on the circuit you're using, not on the (pasta) curves.