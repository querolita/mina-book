# Overview

- To prove things in Mina we use PLONK, or at least a variant of PLONK.
- In this section we explain the variant only (give it a name!)
- It is not a zk-SNARK as it is not succint in the proof size (11kb instead of 1kb?)
- Essentially what PLONK allows you to do is to take a program (with inputs and outputs) and take a snapshot of its execution. Then, you can remove parts of the inputs, but prove to someone that the execution is correct for the remaining inputs and outputs of the snapshot.
- There are a lot of steps involved in the transformation of "I know an input to this program such that when executed with these other inputs it gives this output" to "here's a sequence of operations you can execute to convince you of that".
- The rest of this section will go through the different steps of that ^ + the optimizations + everything else
