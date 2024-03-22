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
  \sum_{k=0}^M b_k z^{-k} X(z) \\
  & =
  b_0 W_0(z) + (b_1 W_1(z) + (b_2 W_2(z) + \cdots + (b_{M=1}
  W_{M-1}(z)+ b_M W_M(z)) \cdots )
  \end{align}
  ```
  where $W_k (z) = z^{-k} X(z)$ for $k=0, 1, \ldots, M$. Clearly,
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

* Below is an example HLS function that implements the direct-form SFG above:
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
  #pragma HLS bind_op variable=y op=mul impl=fabric latency=1
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

## Transposed-form Implementation
* Rewrite {eq}`firz` as follows:
  ```{math}
  :label: fir_transp
  \begin{align}
  Y(z) 
  &= 
  \sum_{k=0}^M b_k z^{-k} X(z) \\
  & =
  b_0 X(z) + z^{-1} (b_1 X(z) + z^{-1} (b_2 X(z) + \cdots +
  z^{-1} (b_{M-1} X(z)+ z^{-1} b_M X(z)) \cdots ).
  \end{align}
  ```
* From the form of {eq}`fir_transp`, we may also get $Y(z)$
  iteratively by defining:
  ```{math}
  :label: fir_transp_u
  \begin{align}
  U_M(z) &= b_M X(z), \\
  U_k(z) &= b_k X(z) + z^{-1} U_{k+1}(z), & k=M-1,M-2,\ldots,0 
  \\
  Y(z) & = U_0(z),
  \end{align}
  ```
  which leads to the following transposed-form SFG:
  \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x} \longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_0}\hspace{9pt}}\!\!\bigcirc\kern-7.5pt\vcenter{\tiny u_0}
    \!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
   & \Big\downarrow  \hspace{25pt}
    \Big\uparrow  {\scriptsize z^{-1}}
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_1}\hspace{9pt}}\!\!\bigcirc\kern-7.5pt\vcenter{\tiny u_1}
    \\[-0pt]
    & \Big\downarrow  \hspace{25pt}
    \Big\uparrow  {\scriptsize z^{-1}} 
    \\[-0pt]
    & \,\vdots \hspace{29pt} \vdots 
    \\[-0pt]
    & \Big\downarrow  \hspace{25pt}
    \Big\uparrow {\scriptsize z^{-1}}
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_M}\hspace{8pt}}\!\!\bigcirc\kern-8.5pt\vcenter{\tiny u_M}
   \end{align*}

* Below is an example HLS function that implements the transposed-form SFG:
  ```c++
  const din_t b[L]={0.5, -0.4, 0.3, -0.2, 0.05, 0.3, -0.3, 0.2, 0.1, -0.1};

  void fir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {

    static dout_t u[L-1] = {};
  #pragma HLS array_partition variable=w type=complete

    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      // Read in new sample from in
      din_t x = in.read();
  #pragma HLS bind_op variable=bx op=mul impl=fabric 
      dout_t bx = b[0]*x;
      // Write to out
      out.write(bx+u[0]);
      
      delay_add_loop: for (int k=1; k<L-1; k++) {
        bx = b[k]*x;
        u[k-1] = bx+u[k];
      }
      bx = b[L-1]*x;
      u[L-2] = bx;
    }
  }
  ```
  VItis HLS gives a RTL implementation with II=1 and a slightly smaller
  latency for the transposed-form SFG.


## Cascade-form Implementation
* Rewrite {eq}`fir` in the cascade form as follows:
  ```{math}
  :label: fir_cascade
  \begin{align}
  Y(z) 
  &= 
  \sum_{k=0}^M b_k z^{-k} X(z) \\
  & =
  b_0 \prod_{k=1}^{M} \left( 1 - z_k z^{-1} \right) X(z)
  \end{align}
  ```
  where $z_1, z_2, \ldots, z_M$ are the zeros of the transfer function
  $H(z) = \sum_{k=0}^M b_k z^{-k}$ of the FIR filter.

* For real-valued filter taps, i.e., all the $b_k$s are real numbers,
  the zeros either are real-valued or come in conjugate pairs. Hence,
  we may rewrite {eq}`fir_cascade` further in the following form:
  ```{math}
  :label: fir_cascade_sos
  \begin{align}
  Y(z) 
  &=
  b_0 \prod_{k=1}^{K} \left( 1 + b_{k,1} z^{-1} + b_{k,2} \right) X(z)
  \end{align}
  ```
  
  where the FIR filter is decomposed into a cascade of $K$ components
  (sections), each is (at most) a second-order FIR filter with
  real-valued taps ($b_{k,1}$ and $b_{k,2}$).

* Each second-order section (SoS) may be implemented in the direct form
  or in the transposed form. For example:
  - Cascade-form SFG with direct-form SoSs:
  \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x}
  \xrightarrow{\hspace{8pt}{\scriptsize b_0}\hspace{7pt}}
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
    \!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow \cdots \longrightarrow
     \!\bigcirc\!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\downarrow  \hspace{26pt}
    \Big\uparrow \hspace{9pt}
    {\scriptsize z^{-1}} \Big\downarrow  \hspace{26pt}
    \Big\uparrow \hspace{44pt}
     {\scriptsize z^{-1}} \Big\downarrow  \hspace{26pt}
    \Big\uparrow 
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,1}}\hspace{7pt}}\!\!\bigcirc \hspace{18pt}
    \!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,1}}\hspace{7pt}}\!\!\bigcirc 
    \hspace{17pt} \cdots \hspace{19pt}
    \bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{K,1}}\hspace{6pt}}\!\!\bigcirc
    \\[-0pt]
     {\scriptsize z^{-1}} & \Big\downarrow  \hspace{26pt}
    \Big\uparrow \hspace{9pt}
    {\scriptsize z^{-1}} \Big\downarrow  \hspace{26pt}
    \Big\uparrow  \hspace{44pt}
     {\scriptsize z^{-1}} \Big\downarrow  \hspace{26pt}
    \Big\uparrow 
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,2}}\hspace{7pt}}\!\!\bigcirc \hspace{18pt}
    \!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,2}}\hspace{7pt}}\!\!\bigcirc 
     \hspace{17pt} \cdots \hspace{19pt}
     \bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{K,2}}\hspace{6pt}}\!\!\bigcirc
   \end{align*}

  - Cascade-form SFG with transposed-form SoSs:
  \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x}
  \xrightarrow{\hspace{8pt}{\scriptsize b_0}\hspace{7pt}}
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
    \!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow \cdots \longrightarrow
     \!\bigcirc\!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
   & \Big\downarrow  \hspace{26pt}
    \Big\uparrow  {\scriptsize z^{-1}} \hspace{9pt}
   \Big\downarrow  \hspace{26pt}
    \Big\uparrow  {\scriptsize z^{-1}}
    \hspace{44pt}
   \Big\downarrow  \hspace{26pt}
    \Big\uparrow   {\scriptsize z^{-1}} 
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,1}}\hspace{7pt}}\!\!\bigcirc \hspace{18pt}
    \!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,1}}\hspace{7pt}}\!\!\bigcirc 
    \hspace{17pt} \cdots \hspace{19pt}
    \bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{K,1}}\hspace{6pt}}\!\!\bigcirc
    \\[-0pt]
     & \Big\downarrow  \hspace{26pt}
    \Big\uparrow {\scriptsize z^{-1}} \hspace{9pt}
   \Big\downarrow  \hspace{26pt}
    \Big\uparrow   {\scriptsize z^{-1}} \hspace{44pt}
     \Big\downarrow  \hspace{26pt}
    \Big\uparrow {\scriptsize z^{-1}} 
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,2}}\hspace{7pt}}\!\!\bigcirc \hspace{18pt}
    \!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,2}}\hspace{7pt}}\!\!\bigcirc 
     \hspace{17pt} \cdots \hspace{19pt}
     \bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{K,2}}\hspace{6pt}}\!\!\bigcirc
   \end{align*}

* Below is an example HLS function that implements the cascade-form
  SFG with transposed-form SoSs:
  ```c++
  const din_t b[K][2]={
    { 0.730052030952496,                  0},
    { 0.092026730665386, -0.382681081883717},
    { 0.675184280532296,  1.091802879870597},
    {-1.634523480450109,  0.879258346962745},
    {-0.662739561700072,  0.745724572309314}
  };
  const din_t b0 = 0.5;

  void fir2nd(dout_t &in, dout_t &out, dout_t &w1, dout_t &w2, const din_t b[2]) {
  #pragma HLS inline

    // Write to out
    out = in+w1;
    // Update register
  #pragma HLS bind_op variable=bx op=mul impl=fabric 
    dout_t bx = b[0]*in;
    w1 = bx+w2;
    bx = b[1]*in;
    w2 = bx;
  }

  void fir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {

    dout_t u[K+1];
    static dout_t w[K][2] = {};

    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      u[0] = b0*in.read();
      cascade_loop: for (int k=0; k<K; k++) {
        fir2nd(u[k], u[k+1], w[k][0], w[k][1], &b[k][0]);
      }
      out.write(u[K]);
    }
  }
  ```
   VItis HLS gives a RTL implementation with II=1 and a higher
  latency that both the direct-form and transposed-form SFGs. In
  addition, significantly more LUT resource is needed.
