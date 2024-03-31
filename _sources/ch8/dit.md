# Decimation-in-Time Algorithm

* Let $x[n]$ be of length at most $M=2^{\nu}$. We will use binary
  sequences of length at most $\nu -1$ bits to index "sub-signals"
  generated from $x[n]$. Let $b$ be a binary sequence ($b$ can be a
  null sequence). Then $0b$ and  $1b$ denote
  concatenating a most significant bit to $b$.

* Define $x^{(\nu)}[n] = x[n]$. For $i=\nu-1, \nu-2, \ldots, 0$,
  define iteratively:
  \begin{align*}
  x^{(i)}_{0b}[n] &= x^{(i+1)}_b [2n] \\
  x^{(i)}_{1b}[n] &= x^{(i+1)}_b [2n+1] 
  \end{align*}
  for each $n=0,1,\ldots,2^i-1$ and each binary sequence $b$ of length
  $\nu-1-i$. 

  That is, we break the original signal $x[n]$ into 2
  sub-signals respectively containing the even- and odd-indexed
  samples, each of which is further broken down into 2 sub-signals of
  even- and odd-indexed samples, and so on until there is a single
  sample left in each sub-signal of the last layer ($i=0$).

* Note that the length of $x^{(i)}_{b}[n]$  is $2^i$. 
  Write the $2^i$-point DFT of $x^{(i)}_{b}[n]$ as:
  \begin{align*}
  X^{(i)}_{b,k} &= \sum_{n=0}^{2^i-1} x^{(i)}_{b}[n] w^{kn}_{2^i} &
  k=0,1,\ldots, 2^i-1.
  \end{align*}

* Now, consider the forward DFT formula {eq}`dft`:
  ```{math}
  :label: e:fft_dit1
  \begin{align}
  X_k 
  &= 
  \sum_{n=0}^{2^\nu-1} x^{(\nu)}[n] w^{kn}_{2^\nu}
  \\
  &=
  \sum_{n=0}^{2^{\nu-1}-1} x^{(\nu)}[2n]  w^{k(2n)}_{2^\nu} +
  \sum_{n=0}^{2^{\nu-1}-1} x^{(\nu)}[2n+1]  w^{k(2n+1)}_{2^\nu} 
  \\
  &=
  \sum_{n=0}^{2^{\nu-1}-1} x^{(\nu-1)}_0[n] w^{kn}_{2^{\nu-1}} +
  w^{k}_{2^\nu}  \left(
  \sum_{n=0}^{2^{\nu-1}-1} x^{(\nu-1)}_1[n] w^{kn}_{2^{\nu-1}} 
  \right)
  & k=0,1,\ldots,2^\nu -1
  \end{align}
  ```
  Notice that the two sums in the last expression of {eq}`e:fft_dit1` above are simply 
  the $2^{\nu-1}$-point DFTs. We may rewrite {eq}`e:fft_dit1` in the
  following form:
  ```{math}
  :label: e:fft_dit2
  \begin{align}
  X_k 
  & =
  X^{(\nu-1)}_{0,k} +  w^{k}_{2^\nu} X^{(\nu-1)}_{1,k}
  \\
  X_{k+2^{\nu-1}} 
  &= 
  X^{(\nu-1)}_{0,k} +w^{k+2^{\nu-1}}_{2^\nu} X^{(\nu-1)}_{1,k}
  & k=0,1,\ldots,2^{\nu-1} -1
  \end{align}
  ```
* Repeat the decomposition above recursively, we get, for $i=\nu-1, \nu-2, \ldots, 1$,
  ```{math}
  :label: e:fft_dit3
  \begin{align}
  X^{(i)}_{b,k} 
  & =
  X^{(i-1)}_{0b,k} +  w^{k}_{2^i} X^{(i-1)}_{1b,k}
  \\
  X^{(i)}_{b,k+2^{i-1}} 
  &= 
  X^{(i-1)}_{0b,k} +w^{k+2^{i-1}}_{2^i} X^{(i-1)}_{1b,k}
  \end{align}
  ```
    for each $k=0,1,\ldots,2^{i-1} -1$ and each binary sequence $b$ of
    length $\nu-i$ until reaching the trivial terminating case:
    ```{math}
  :label: e:fft_dit4
  \begin{align}
  X^{(1)}_{b,0} 
  &=
  X^{(0)}_{0b,0} + w^{0}_{2} X^{(0)}_{1b,0}
  =
  x^{(0)}_{0b}[0] + x^{(0)}_{1b}[0]
  \\
  X^{(1)}_{b,1} 
  &= 
  X^{(0)}_{0b,0} + w^{1}_{2} X^{(0)}_{1b,0}
  =
  x^{(0)}_{0b}[0] - x^{(0)}_{1b}[0]
  \end{align}
  ```
  for each binary sequence $b$ of length $\nu-1$. 
  Note that in {eq}`e:fft_dit4`, for each binary sequence $b$ of
  length $\nu$, $x^{(0)}_{b}[0] = x[b]$ where $b$ in $x[b]$ should be
  interpreted as the standard binary representation of the time index
  $n$ in $x[n]$. For example, if $\nu = 4$, $x^{(0)}_{1000}[0] = 
  x[1000] = x[8]$.

* The radix-2 decimation-in-time FFT algorithm amounts to starting
  from {eq}`e:fft_dit4`, working up each *stage* using
  {eq}`e:fft_dit3`, and finally obtaining the DFT coefficients $X_k$'s
  using {eq}`e:fft_dit2`. 

* It is easy to see that going up each stage $i=0, 1, \ldots, \nu -1$,
  we need to calculate and store the intermediate values
  $X^{(i)}_{b,k}$ generated in the FFT algorithm, requiring:
  - $2^{\nu}$ complex-valued storage units, and
  - at most $2^{\nu}$ complex-valued multiplications and $2^{\nu}$
    complex-valued additions.

  Hence, both the computational complexity and the storage requirement
  are $\mathcal{O}(\nu 2^{\nu}) = \mathcal{O}(M \log_2 M)$. For
  comparison, recall that the computation complexity of directly using
  the forward DFT formula {eq}`dft` to calculate the DFT is
  $\mathcal{O}(M^2)$ and that the storage requirement is
  $\mathcal{O}(M)$. **Thus, the FFT algorithm provides a significant
  reduction in the computational complexity, requiring a relatively
  small increase in the storage requirement.**
