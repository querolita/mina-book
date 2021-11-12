# ChaCha20

This gate uses plookup.

Each line has the form

$$x += z; y ^= x; y <<<= k$$

or without mutation,

$$x' = x + z; y' = (y ^ x') <<< k$$

![](../../img/chacha1.png)

![](../../img/chacha2.png)