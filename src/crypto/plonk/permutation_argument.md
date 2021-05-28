# Permutation argument

**WARNING: this section is bad, it's incorrect, it should be re-written with a similar incremental approach though**.

## Copy constraints

TODO: link to relevant section, would be nice to have a diagram visually showing that copy constraints are ensuring that the different wires of different gates are actually connected

Imagine $a_1 = b_2$.

## Copy constraints reduced to a permutation

Copy constraints can be reduced to show that:

$$ assignments = permutation(assignments) $$

That is, if $a_1$ must be equal to $b_2$ we would want to show that these two ordered sets (is ordered set the right term?) are equal:

$$ (a_1, a_2, b_1, b_2, c_1, c_2) = (b_2, a_2, b_1, a_1, c_1, c_2) $$

To be more formal, we can write the permutation as:

$$ \sigma = (a_1 b_2) $$

## Permutation to an accumulator

As usual, the end goal is to reduce our constraint to a polynomial, so that we can prove it using our polynomial commitment techniques.

We can reduce our constraint to another one, showing that the two following sequences are:

1. well-formed and 
2. ends up accumulating the same result

here is one of the sequences: 

$$

\begin{cases}
acc_0 = 1\\
acc_i = acc_{i-1} \cdot a_{i} \text{ for } i \in [[1, n]] \\
acc_i = acc_{i-1} \cdot b_{i-n} \text{ for } i \in [[n+1, 2n]] \\
acc_i = acc_{i-1} \cdot c_{i-2n} \text{ for } i \in [[2n+1, 3n]] \\
\end{cases}

$$

and the last term is: 

$$ acc_{n+1} = 1 \times a_1 \times \cdots \times a_n \times b_1 \times \cdots \times b_n \times c_1 \times \cdots \times c_n$$

which accumulates all the assignments.

The other sequence is an accumulation of the permuted stuff:

$$

\begin{cases}
acc_0 = 1\\
acc_i = acc_{i-1} \cdot a_{\sigma(i)} \text{ for } i \in [[1, n]] \\
acc_i = acc_{i-1} \cdot b_{\sigma(i-n)} \text{ for } i \in [[n+1, 2n]] \\
acc_i = acc_{i-1} \cdot c_{\sigma(i-2n)} \text{ for } i \in [[2n+1, 3n]] \\
\end{cases}

$$


which should accumulate to the same final term as well (just in a different order), and where $\sigma$ is defined as:

$$
\begin{cases}
\sigma(1) = n+2\\
\sigma(n+2) = 1\\
\sigma(i) = i \text{ for all other } i
\end{cases}
$$

in order to enforce $a_1 = b_2$.

this doesn't work though, as plenty of terms can cancel one another...

But it works if we take a random 


## Shortening the size of the accumulator

We can also divide one sequence with the other, to get 1 on the final term:

$$
\begin{cases}
acc_0 = 1\\
acc_i = acc_{i-1} \cdot f_i / g_i
\end{cases}
$$

wait, could we have done that as well?

$$
\begin{cases}
acc_0 = 1\\
acc_i = acc_{i-1} + f_i - g_i
\end{cases}
$$

and the end term would have been 1 also? (or $0$ if $acc_0 = 0$)

## Reducing the size of the accumulator

We are listing all the $a_i$ and $b_i$ and $c_i$ sequentially. Since they are the same size, we might ask: can we shorten that list?
And yes we can.

$$
\begin{cases}
acc_0 = 1\\
acc_i = acc_{i-1} \cdot a_{i} \cdot b_{i} \cdot c_{i}
\end{cases}
$$

but this won't work as they can cancel one another

## Enforcing that unexpected terms cancel one another

We add $\beta$ and $\gamma$ challenges by the verifier:

$$ 
\begin{cases}
acc_0 = 1\\
acc_i = acc_{i-1} (\frac{(a_i + \beta \omega^{i-1} + \gamma)(b_i + \beta k_1 \omega^{i-1} + \gamma)(c_i + \beta k_2 \omega^{i-1} + \gamma)}{(a_i + \beta \sigma_1(\omega^{i-1}) + \gamma)(b_i + \beta \sigma_2(\omega^{i-1}) + \gamma)(c_i + \beta \sigma_3(\omega^{i-1}) + \gamma)})
\end{cases}
$$


## Accumulator to polynomial

After finding the terms of $acc_0, acc_1, \cdots, acc_{3n}$, we can interpolate the polynomial $acc(x)$ of degree $3n$.


## Zero-knowledge

Finally, generate random $b_7, \cdots, b_9 \in \mathbb{F}$ in order to hide the polynomial:

$$ (b_7 x^2 + b_8 x + b_9) Z_H(x) + acc(x) $$

TODO: should this step only be part of the final protocol?
