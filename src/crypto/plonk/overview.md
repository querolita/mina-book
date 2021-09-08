# Overview

The proof system in Mina is a variant of [PLONK](). To understand PLONK, you can refer to our [series of videos](https://www.youtube.com/watch?v=RUZcam_jrz0&list=PLBJMt6zV1c7Gh9Utg-Vng2V6EYVidTFCC) on the scheme.
In this section we explain our variant, called **15-wires**.
<!-- TODO: embed each video in their respective category -->

15-wires is not formally a zk-SNARK, as it is not succinct in the proof size. zk-SNARKs must a $log(n)$ proof size where n is the number of gates in the circuit. (In practice, our proofs are in the order of dozens of kilobytes).

Note that 15-wires is a zk-SNARK in terms of verification complexity ($log(n$)) and proving complexity: it is linear. The prover can't be faster than linear, as they at least have to read the circuit.

Essentially what PLONK allows you to do is to, given a program with inputs and outputs, take a snapshot of its execution. Then, you can remove parts of the inputs, or outputs, or execution trace, and still prove to someone that the execution was performed correctly for the remaining inputs and outputs of the snapshot.
There are a lot of steps involved in the transformation of "I know an input to this program such that when executed with these other inputs it gives this output" to "here's a sequence of operations you can execute to convince yourself of that".
