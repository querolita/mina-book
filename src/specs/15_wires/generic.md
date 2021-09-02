# Generic

The generic gate is the normal PLONK gate, which allows us to implement a number of different gates depending on how it's called:

* addition gate: the first three columns are used to represent the left, right, and output wire
* multiplication gate: TKTK
* constant gate: only the most-left column is set to what the constant value should be
* public input gate: only the most-left column is set
