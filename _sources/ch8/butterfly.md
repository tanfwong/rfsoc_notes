# Butterfly Structures

## Radix-2 Decimation-in-Time Butterfly
* The radix-2 decimation-in-time FFT algorithm in
  {eq}`e:fft_dit2`-{eq}`e:fft_dit4` is perhaps more commonly described
  by the *butterfly-structured* SFG showing how to obtain the
  $M$-point DFT coefficients $X_0, X_1, \ldots, X_{M-1}$ from the $M$
  signal samples $x[n]$ for $n=0,1,\ldots, M-1$. 

* For example, the figure below shows the decimation-in-time butterfly
  SFG for the case of $M=2^3 = 8$ ($\nu = 3$): 
  ```{figure} ../figs/fft_dit_bfly.jpg 
  ---
  name: butterfly8
  alt: 8-point decimation-in-time FFT butterfly 
  width: 800px
  align: center
  ---
  Eight-point radix-2 decimation-in-time FFT butterfly SFG
  ``` 
  The figure also shows calculation of the $8$-point FFT is decomposed
  into that of two $4$-point FFTs, each of which is further decomposed
  into that of two $2$-point FFTs recursively as described by
  {eq}`e:fft_dit2`-{eq}`e:fft_dit4`.

* We see from the butterfly SFG above that the input $x[n]$ to the
  butterfly SFG is shuffled (decimated) according to the binary pattern
  $b$ in the $i=0$ stage. The **bit reversal** process (from MSB to
  LSB) provides a simple mnemonic to establish the order of shuffling.

* Rewriting the IDFT formula {eq}`idft` in the form below
  \begin{equation*}
  M x[n] = \sum_{k=0}^{M-1} X_k (w^{kn}_M)^*
  \end{equation*}
  indicates that the same butterfly SFG above can be used to calculate
  IDFT if one replaces:
  1. the butterfly input (still needs to shuffle the order)
     with $X_k$,
  2. the butterfly output with $Mx[n]$, and
  3. all gain factors with their respective complex conjugates.

## Modified Butterfly for Implementation
* The butterfly SFG in {numref}`butterfly8` can be further modified to
  make it more conducive to implementation in the PL. To see how, let
  us get back to {eq}`e:fft_dit3` which gives the following SFG for
  the basic butterfly element in {numref}`butterfly8`:
  ```{figure} ../figs/butterfly2.jpg 
  ---
  name: butterfly2
  alt: Basic 2-point butterfly element 
  width: 400px
  align: center
  ---
  Basic 2-point butterfly element SFG in {eq}`e:fft_dit3`
  ```
  The SFG implies that 2 complex-valued multiplications and 2
  complex-valued additions are needed to implement this basic element.

* However, it is easy to see that the computational requirement can
  actually be lowered by rewriting {eq}`e:fft_dit3` using the
  "fraction-like" arithmetic of $w^k_M$ as follows:
  ```{math}
  :label: btfly2mod
  \begin{align}
  X^{(i)}_{b,k} 
  & =
  X^{(i-1)}_{0b,k} +  w^{k2^{\nu-i}}_{M} X^{(i-1)}_{1b,k}
  \\
  X^{(i)}_{b,k+2^{i-1}} 
  &= 
  X^{(i-1)}_{0b,k} - w^{k2^{\nu-i}}_{M} X^{(i-1)}_{1b,k}
  \end{align}
  ```
  for each $i=\nu, \nu-1, \ldots, 1$, $k=0,1,\ldots,2^{i-1} -1$, and
    each binary sequence $b$ of length $\nu-i$.

* Expressing {eq}`btfly2mod` as a SFG, we obtain the modified basic
  butterfly element as shown below:
  ```{figure} ../figs/butterfly2_mod.jpg 
  ---
  name: butterfly2_mod
  alt: Modified basic 2-point butterfly element 
  width: 500px
  align: center
  ---
  Modified basic 2-point butterfly element SFG in {eq}`btfly2mod`
  ```
  We see that implementation of the modified SFG requires only a
  single complex-valued multiplication, addition, and subtraction
  each.
  
* Replacing each basic element in the 8-point butterfly SFG in
  {numref}`butterfly8` with the modified element in
  {numref}`butterfly2_mod`, we obtain the following modified 8-point
  butterfly SFG:
  ```{figure} ../figs/butterfly8_mod.jpg 
  ---
  name: butterfly8_mod
  alt: modified 8-point decimation-in-time FFT butterfly 
  width: 1000px
  align: center
  ---
  Modified 8-point radix-2 decimation-in-time FFT butterfly SFG
  ``` 
  which is more conducive to PL implementation. Note that the first
  stage ($i=0$) does not require a gain layer because $w^0_M = 1$ (or
  see {eq}`e:fft_dit4`). 
