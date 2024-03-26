# IIR Filter
* Consider an $N$-order IIR filter specified by {eq}`iir` in the time
domain or {eq}`iirz` in the $z$-domain. For the sake of drawing
simpler SFGs, we assume below that the feedback order $N$ is at least
as large as the feedforward order $M$, i.e., $N \geq M$.

* The transfer function $H(z)$ of the IIR filter is given by 
  ```{math}
  :label: iir_tf 
  \begin{align} H(z) 
  &= \frac{\sum_{k=0}^M b_k z^{-k}}{\sum_{k=0}^N a_k z^{-k}}
  & ~~~(a_0=1).
  \end{align} 
  ```
  Thus, the IIR filter can be interpreted as a cascade of a
  *feedforward component*, i.e., an $M$-order FIR filter with transfer
  function $\sum_{k=0}^M b_k z^{-k}$, and a *feedback component*,
  i.e., an $N$-order IIR filter with transfer function
  $\frac{1}{\sum_{k=0}^N a_k z^{-k}}$ and a trivial feedforward tap.

## Direct-form Implementation
* The feedback component, i.e.,
  \begin{equation*} 
  Y(z) = - \sum_{k=1}^N a_k z^{-k} Y(z), 
  \end{equation*} 
  can be implemented in the direct form similar to the development in
  {numref}`sec:fir_direct`.
  
* When the feedforward FIR component is also implemented in the direct
  form, we obtain the following SFG of direct form I for the IIR
  filter:
   \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x} \longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_0}\hspace{9pt}}\!\!\bigcirc\!\!\xrightarrow{\hspace{26pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{26pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow \hspace{27pt}
    \Big\uparrow  \hspace{26pt}
    \Big\downarrow  {\scriptsize z^{-1}}
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_1}\hspace{9pt}}\!\!\bigcirc \hspace{25.5pt}
    \!\bigcirc\!\!\xleftarrow{\hspace{7pt}{\scriptsize
    -a_1}\hspace{6pt}}\!\!\bigcirc
    \\[-0pt]
     {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow \hspace{27pt}
    \Big\uparrow  \hspace{26pt}
    \Big\downarrow  {\scriptsize z^{-1}}
    \\[-0pt]
    & \,\vdots \hspace{29pt} \vdots  \hspace{29pt}
    \,\vdots \hspace{30pt} \vdots 
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow \hspace{27pt}
    \Big\uparrow  \hspace{26pt}
    \Big\downarrow  {\scriptsize z^{-1}}
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_M}\hspace{8pt}}\!\!\bigcirc \hspace{25pt}
    \!\bigcirc\!\!\xleftarrow{\hspace{6pt}{\scriptsize
    -a_N}\hspace{5pt}}\!\!\bigcirc
   \end{align*}
 
* Swapping the order of the feedforward and feedback components in the
  cascade above, we obtain following SFG of direct form II for the IIR
  filter:
   \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x} \longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!\bigcirc
    \!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_0}\hspace{9pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow 
    \\[-10pt]
     &  \!\bigcirc\!\!\xleftarrow{\hspace{6.5pt}{\scriptsize
    -a_1}\hspace{7pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_1}\hspace{9pt}}\!\!\bigcirc 
    \\[-0pt]
     & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow 
    \\[-0pt]
    & \,\vdots \hspace{30pt} \vdots  \hspace{27.5pt}
    \,\vdots 
    \\[-0pt]
     & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow 
    \\[-10pt]
     &\!\bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_M}\hspace{5pt}}
     \!\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_M}\hspace{8pt}}\!\!\bigcirc 
    \\[-0pt]
     & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} 
    \\[-0pt]
    & \,\vdots \hspace{30pt} \vdots 
    \\[-0pt]
     & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} 
    \\[-10pt]
     &\!\bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_N}\hspace{5.5pt}}\!\!\bigcirc
   \end{align*} 

* Below is an example HLS function that implements the direct-form II
  SFG above:
  ```c++
  // This is a Chebyshev type-II lowpass filter with M=N=6 (L=7) 
  const din_t b[ffL] = {
    0.168814352044553, 0.792090023955804, 1.723851142494651, 2.197013405545998,
    1.723851142494651, 0.792090023955804, 0.16881435204455
  };

  const din_t a[fbL] = {
    1.0, 1.776425956077127, 2.196211818205448, 1.583727976930950, 
    0.765387107901598, 0.216231507219810, 0.028540076201079
  };

  void iir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {

    static din_t w[fbL] = {};
  #pragma HLS array_partition variable=w type=complete

    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      delay_loop: for (int k=fbL-1; k>0; k--) {
        w[k] = w[k-1];
      }
      // Read in new sample from in
      dout_t y = in.read();
      dout_t by = 0.0;
  #pragma HLS bind_op variable=y op=mul impl=fabric
  #pragma HLS bind_op variable=by op=mul impl=fabric
      acc_loop: for (int k=1; k<fbL; k++) {
        y -= a[k]*w[k];
        if (k<ffL)
          by += b[k]*w[k];
      }
      w[0] = y;
      y *= b[0];

      // Write to out
      out.write(y+by);
    }
  }
  ```
    - Vitis HLS gives a RTL implementation of `sample_loop` with
      II=2.
    - The bottleneck prevents achieving II=1 is the *carried
      dependence that updating `w[0]` in an iteration of `sample_loop`
      requires the accumulation of `y` in `acc_loop` to complete
      first. Although `acc_loop` is automatically unrolled by Vitis
      HLS, it still takes at least two clock cycles to complete
      accumulation on `y` for the example IIR filter of order $6$.  As
      a result, II=1 cannot be achieved for this filter.
    - This bottleneck problem is inherent to the direct-form structure
      of the IIR implementation, and would be more severe as the order
      of the IIR filter increases. Thus, the direct-from
      implementation may not be suitable for implementing higher-order
      IIR filters operating at a high sampling rate.


## Transposed-form Implementation
* When both the feedforward and feedback components in the cascade are
  implemented in the transposed form similar to the development in
  {numref}`sec:fir_transp`, we obtain the following SFG of transposed
  from I:

   \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x} \longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!\bigcirc
    \!\!\xrightarrow{\hspace{26pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_0}\hspace{9pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\uparrow  \hspace{25pt}
    \Big\downarrow \hspace{27pt}
    \Big\downarrow  \hspace{26pt}
    \Big\uparrow  {\scriptsize z^{-1}}
    \\[-10pt]
     &\!\bigcirc\!\!\xleftarrow{\hspace{6pt}{\scriptsize
    -a_1}\hspace{6pt}}\!\!\bigcirc \hspace{25.5pt}
    \!\bigcirc\!\!\xrightarrow{\hspace{9pt}{\scriptsize
    b_1}\hspace{9pt}}\!\!\bigcirc
    \\[-0pt]
     {\scriptsize z^{-1}} & \Big\uparrow  \hspace{25pt}
    \Big\downarrow \hspace{27pt}
    \Big\downarrow  \hspace{26pt}
    \Big\uparrow  {\scriptsize z^{-1}}
    \\[-0pt]
    & \,\vdots \hspace{29pt} \vdots  \hspace{29pt}
    \,\vdots \hspace{30pt} \vdots 
    \\[-0pt]
    {\scriptsize z^{-1}} & \Big\downarrow  \hspace{25pt}
    \Big\uparrow \hspace{27pt}
    \Big\uparrow  \hspace{26pt}
    \Big\downarrow  {\scriptsize z^{-1}}
    \\[-10pt]
     &\!\bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_N}\hspace{4.5pt}}\!\!\bigcirc \hspace{25pt}
    \!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_M}\hspace{9pt}}\!\!\bigcirc
   \end{align*}

* Swapping the order of the feedforward and feedback components in the
  cascade above, we obtain following SFG of transposed form II for the IIR
  filter:
   \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x} \longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{10pt}{\scriptsize
    b_0}\hspace{9pt}}\!\!\bigcirc
    \!\!\xrightarrow{\hspace{24pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow 
    \\[-10pt]
     &  \!\bigcirc\!\!\xrightarrow{\hspace{10pt}{\scriptsize
    b_1}\hspace{9pt}}
    \!\!\bigcirc\!\!\xleftarrow{\hspace{6.5pt}{\scriptsize
    -a_1}\hspace{6pt}}\!\!\bigcirc 
    \\[-0pt]
     & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow 
    \\[-0pt]
    & \,\vdots \hspace{30pt} \vdots  \hspace{27.5pt}
    \,\vdots 
    \\[-0pt]
     & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow 
    \\[-10pt]
     &\!\bigcirc\!\!\xrightarrow{\hspace{8.5pt}{\scriptsize
    b_M}\hspace{8pt}}
     \!\!\bigcirc\!\!\xleftarrow{\hspace{5pt}{\scriptsize
    -a_M}\hspace{5pt}}\!\!\bigcirc 
    \\[-0pt]
    & \hspace{33pt} \Big\uparrow   {\scriptsize z^{-1}} 
    \hspace{15pt} \Big\downarrow
    \\[-0pt]
    & \hspace{33pt} \,\vdots \hspace{29.5pt} \vdots 
    \\[-0pt]
    & \hspace{33pt} \Big\uparrow   {\scriptsize z^{-1}} 
    \hspace{15pt} \Big\downarrow
    \\[-10pt]
     & \hspace{33pt} \!\bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_N}\hspace{5.5pt}}\!\!\bigcirc
   \end{align*} 

* Below is an example HLS function that implements the transposed-form II
  SFG above:
  ```c++
  // This is a Chebyshev type-II lowpass filter with M=N=6 (L=7) 
  const din_t b[ffL] = {
    0.168814352044553, 0.792090023955804, 1.723851142494651, 2.197013405545998,
    1.723851142494651, 0.792090023955804, 0.16881435204455
  };

  const din_t a[fbL] = {
    1.0, 1.776425956077127, 2.196211818205448, 1.583727976930950, 
    0.765387107901598, 0.216231507219810, 0.028540076201079
  };

  void iir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {

    dout_t u[fbL-1] = {};
  #pragma HLS array_partition variable=u type=complete
  
    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      din_t x = in.read();
      dout_t bx = b[0]*x;
      dout_t y = bx+u[0];
      dout_t ay;
  #pragma HLS bind_op variable=bx op=mul impl=dsp
  #pragma HLS bind_op variable=ay op=mul impl=dsp
      // Write to out
      out.write(y);

      delay_add_loop: for (int k=1; k<fbL-1; k++) {
        ay = a[k]*y;
        if (k<ffL) {
          bx = b[k]*x;
          u[k-1] = u[k]+bx-ay;
        } else {
          u[k-1] = u[k]-ay;
        }
      }
      ay = a[fbL-1]*y;
      if (fbL==ffL) {
        bx = b[fbL-1]*x;
        u[fbL-2] = bx-ay;
      } else {
        u[fbL-2] = -ay;
      } 
    }
  }
  ```
  - Vitis HLS gives a RTL implementation of `sample_loop` with
      II=1.
  - The structure of the transposed-form implementation "simplifies"
    the carried dependence in `sample_loop` to that the update
    `y=bx+u[0]` needs to complete before the calculation of `a[1]*y`
    for updating `u[0]` in the next iteration of the loop. Similar to
    the first-order IIR filter example in {numref}`sec:loop_pipe`, the
    objective of II=1 can be achieved as long as the update
    `y=bx+u[0]` and the multiplication `a[1]*y` afterward can be
    achieved within a clock cycle. This condition is much easier to
    satisfied for example by using DSP slices to implement the
    multipliers.
  - In general, the transposed-form structure is more suitable
     for higher speed implementation of IIR filters since the carried
     dependence induced by the transposed-form structure involves only
     a single multiplication rather than the accumulation of many
     products.

