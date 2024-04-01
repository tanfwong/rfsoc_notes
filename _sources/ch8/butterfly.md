# Butterfly Structures

* The radix-2 decimation-in-time FFT algorithm in
  {eq}`e:fft_dit2`-{eq}`e:fft_dit4` is perhaps more commonly described
  by the *butterfly-structured* SFG showing how to obtain the
  $M$-point DFT coefficients $X_0, X_1, \ldots, X_{M-1}$ from the $M$
  signal samples $x[n]$ for $n=0,1,\ldots, M-1$. 

* For example, the figure below shows the decimation-in-time butterfly
  SFG for the case of $M=2^3 = 8$ ($\nu = 3$): 
  ```{image} ../figs/fft_dit_bfly.jpg 
  :alt: 8-point decimation-in-time FFT butterfly 
  :width: 800px 
  :align: center 
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
