# Fast Fourier Transform

* The term **Fast Fourier Transform (FFT)** usually is used to
  describe a general class of computationally efficient algorithms to
  calculate discrete Fourier transform (DFT) and its inverse (IDFT) of
  a finite-length discrete-time signal.

* The $M$-point (forward) DFT of a signal $x[n]$ of length at most $M$ (i.e.,
  $x[n] = 0$ for $n \neq 0,1, \ldots, M-1$)  is:
  ```{math}
  :label: dft
  \begin{align}
  X_k &= \sum_{n=0}^{M-1} x[n] w^{kn}_{M} & k=0,1,2,
  \ldots, M-1
  \end{align}
  ```
  and its $M$-point IDFT gives back the signal $x[n]$:
  ```{math}
  :label: idft
  \begin{align}
  x[n] &= \frac{1}{M} \sum_{k=0}^{M-1} X_k w^{-kn}_{M} &
  n=0,1,2, \dots, M-1
  \end{align}
  ```
  where $\displaystyle w^k_M = e^{-j\frac{2\pi k}{M}}$. 

* It is not hard to verify that the reduction rule for any product of
  terms in the form of $w^k_M$ is the same as that for a sum of the
  fractions $\frac{k}{M}$ modulo $1$ (i.e., the reduction rule for the
  sum fraction removing any whole number part). For example, $w^k_M
  \cdot w^{k'}_{M'} = w^l_N$ where $\frac{l}{N} = \left(\frac{k}{M} +
  \frac{k'}{M'} \right) \bmod 1$.

* Direct calculation of DFT and IDFT using {eq}`dft` and {eq}`idft` is
  computationally inefficient, requiring $\mathcal{O}(M^2)$
  complex-valued multiplications and $\mathcal{O}(M^2)$ complex-valued
  additions.

* The main idea behind any FFT algorithm is to look for repetitive
  patterns in the calculation of DFT/IDFT and store results of
  calculations that can be repeatedly reused later to reduce the total
  amount of calculations needed. In this sense, FFT trades
  computational (or time) complexity against storage complexity.

* For simplicity, we focus only on the *radix-2 decimation-in-time
  FFT algorithm* in this section. 
