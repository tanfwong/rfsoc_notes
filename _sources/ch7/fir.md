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
* Combining {eq}`fir_direct_w` and {eq}`fir_direct_u` gives the
  following direct-form SFG:
  \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x} \longrightarrow
    &
    \!\bigcirc\kern-7.5pt\vcenter{\tiny w_0}\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_0}\hspace{9pt}}\!\!\bigcirc\kern-7.5pt\vcenter{\tiny u_0}
    \!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow
    \\[-10pt]
     &\!\bigcirc\kern-7.5pt\vcenter{\tiny
  w_1}\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_1}\hspace{9pt}}\!\!\bigcirc\kern-7.5pt\vcenter{\tiny u_1}
    \\[-0pt]
     {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow
    \\[-0pt]
    & \,\vdots \hspace{29pt} \vdots 
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow
    \\[-10pt]
     &\!\bigcirc\kern-8.5pt\vcenter{\tiny
  w_M}\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_M}\hspace{8pt}}\!\!\bigcirc\kern-8.5pt\vcenter{\tiny u_M}
   \end{align*}

* Example HLS function that implements the direct-form SFG above:
  ```c++
  #define L 10
  const din_t b[L]={0.5, -0.4, 0.3, -0.2, 0.05, 0.3, -0.3, 0.2, 0.1, -0.1};

  void fir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {

    static din_t w[L] = {};
  #pragma HLS array_partition variable=w type=complete
    
    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      
      delay_loop: for (int k=L-1; k>0; k--) {
        w[k] = w[k-1];
      }
      // Read in new sample from in
      w[0] = in.read();

      dout_t y = 0.0;
      acc_loop: for (int k=0; k<L; k++) {
        y += b[k]*w[k];
      }
    }

    // Write to out
    out.write(y);
  }
  ```
  Vitis HLS pipelines `fir_loop` to achieve II=1 for the function `fir()`.
  ```{tip}
  The `static` qualifier in the declaration of the array `w[L]`
  prevents Vitis HLS from synthesizing hardware to reinitialize
  the contents of the array every time the
  function is called. This allows filtering contiguous blocks of
  samples of length `N`.
  ```

  
