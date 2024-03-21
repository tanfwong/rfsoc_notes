# FIR Filter 
Consider an $M$-order FIR filter specified by {eq}`fir` in the time
domain or {eq}`firz` in the $z$-domain:

## Direct-form Implementation
* Rewrite {eq}`firz` as follows:
  ```{math}
  :label: fir_direct1
  \begin{align}
  Y(z) 
  &= 
  \sum_{k=0}^M b_k z^k X(Z) \\
  & =
  b_0 W_0(z) + (b_1 W_1(z) + (b_2 W_2(z) + \cdots + (b_{M=1}
  W_{M-1}(z)+ b_M W_M(z)) \cdots )
  \end{align}
  ```
  where $W_k (z) = z^k X(z)$ for $k=0, 1, \ldots, M$. Clearly,
   ```{math}
  :label: fir_direct_w
  \begin{align}
  W_k(z) &= z^{-1} W_{k-1}(z), & k=1,2,\ldots, M.
  \end{align}
  ```

* From the form of {eq}`fir_direct1`, we may also get $Y(z)$
  iteratively by defining:
  ```{math}
  :label: fir_direct_u
  \begin{align}
  U_M(z) &= b_M W_M(z), \\
  U_k(z) &= b_k W_{k}(z) + U_{k+1}(z), & k=M-1,M-2,\ldots,0 
  \\
  Y(z) & = U_0(z).
  \end{align}
  ```
