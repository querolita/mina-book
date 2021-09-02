# Poseidon hash

**THIS IS WORK-IN-PROGRESS**

A hash function that is efficient for zk-SNARKs. It is based on the sponge function, with a state composed of field elements and a permutation based on field element operation (addition and exponentiation).

The perumtation is built like SPN block ciphers, with an S-box (exponentiation a group element), adding constants to the state, and matrix multiplication of the state (multiplications and additions) with an MDS matrix.

Since a field element is around 255-bit, a single field element is enough as the capacity of the sponge. The state is therefore often small, with our state being 4 field elements and a rate of 3 field elements.

* main website https://www.poseidon-hash.info/
* our ocaml implementation: https://github.com/minaprotocol/mina/blob/develop/src/lib/random_oracle/random_oracle.mli
* relies on random_oracle_input: https://github.com/minaprotocol/mina/blob/develop/src/lib/random_oracle_input/random_oracle_input.ml
* is instantiated with two types of fields:
    - https://github.com/minaprotocol/mina/blob/develop/src/nonconsensus/snark_params/snark_params_nonconsensus.ml
    - pickles: 
        + seems to rely on zexe code (https://www.youtube.com/watch?v=RItcNRChrzI&t=1732s)

we currently have a few choices:

* specify our own version
* adhere to the [zcash poseidon spec](https://github.com/C2SP/C2SP/pull/3)
* specify an extension of the zcash poseidon spec
