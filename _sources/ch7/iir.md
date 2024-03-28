# IIR Filter
* Consider an $N$-order IIR filter specified by {eq}`iir` in the time
domain or {eq}`iirz` in the $z$-domain. For the sake of drawing
simpler SFGs, we assume below that the feedback order $N$ is at least
as large as the feedforward order $M$, i.e., $N \geq M$.

* The transfer function $H(z)$ of the IIR filter is given by 
  ```{math}
  :label: iir_tf 
  \begin{align} 
  H(z) 
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

    static dout_t u[fbL-1] = {};
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

## Cascade-form Implementation
* As in {numref}`sec:fir_cascade`, if the IIR filter taps in
  {eq}`iir_tf` are all real-valued, then the transfer function in
  {eq}`iir_tf` of the IIR filter can be factored into a product of the
  transfer functions of SoSs with real-valued taps in the form below:
    ```{math}
  :label: iir_sos
  \begin{align} 
  H(z) 
  &= b_0 \prod_{k=1}^K 
  \frac{1+b_{k,1} z^{-1} + b_{k,2} z^{-2}}{1+ a_{k,1} z^{-1} + a_{k,2} z^{-2}}
  \end{align} 
  ```
  Hence, the IIR filter may be implemented as a cascade of all the
  SoSs. 

* Each SoS may be implemented in the direct form II or in the
  transposed form II. For example:
  - Cascade-form SFG with direct-form II SoSs:
  \begin{align*}
    \!\bigcirc\kern-6.5pt\vcenter{\tiny x}
  \xrightarrow{\hspace{8pt}{\scriptsize b_0}\hspace{7pt}}
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{24.5pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
    \!\!\xrightarrow{\hspace{25pt}}\!\!\bigcirc
    \!\!\xrightarrow{\hspace{24.5pt}}\!\!\bigcirc
    \!\!\longrightarrow \cdots \longrightarrow
    \!\bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow \hspace{19pt}
    \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow \hspace{54pt}
    \Big\uparrow \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow 
    \\[-10pt]
    & \!\bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_{1,1}}\hspace{4pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,1}}\hspace{7pt}}\!\!\bigcirc \hspace{15pt}
    \bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_{2,1}}\hspace{4pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{7pt}{\scriptsize
    b_{2,1}}\hspace{7pt}}\!\!\bigcirc 
    \hspace{17pt} \cdots \hspace{19pt}
    \bigcirc\!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -a_{K,1}}\hspace{3pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{6.5pt}{\scriptsize
    b_{K,1}}\hspace{6pt}}\!\!\bigcirc
    \\[-0pt]
    & \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow \hspace{19pt}
    \Big\uparrow  \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow \hspace{54.5pt}
    \Big\uparrow \hspace{26pt}
   \Big\downarrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\uparrow 
    \\[-10pt]
     & \!\bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_{1,2}}\hspace{4pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,2}}\hspace{7pt}}\!\!\bigcirc \hspace{15.5pt}
    \bigcirc\!\!\xleftarrow{\hspace{5.5pt}{\scriptsize
    -a_{2,2}}\hspace{4pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,2}}\hspace{7pt}}\!\!\bigcirc 
     \hspace{17pt} \cdots \hspace{18pt}
    \bigcirc\!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -a_{K,2}}\hspace{3pt}}
    \!\!\bigcirc\!\!\xrightarrow{\hspace{6.5pt}{\scriptsize
    b_{K,2}}\hspace{6pt}}\!\!\bigcirc
   \end{align*}

  - Cascade-form SFG with transposed-form II SoSs:
  \begin{align*}
  \!\bigcirc\kern-6.5pt\vcenter{\tiny x}
    \xrightarrow{\hspace{8pt}{\scriptsize b_0}\hspace{7pt}}
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{24.5pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
    \!\!\xrightarrow{\hspace{25pt}}\!\!\bigcirc
    \!\!\xrightarrow{\hspace{24.5pt}}\!\!\bigcirc
    \!\!\longrightarrow \cdots \longrightarrow
    \!\bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{25pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
    \\[-0pt]
    & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow \hspace{19pt}
    \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow \hspace{54pt}
    \Big\downarrow \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow 
    \\[-10pt]
    & \!\bigcirc
    \!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,1}}\hspace{7.5pt}}\!\!\bigcirc 
    \!\!\xleftarrow{\hspace{4pt}{\scriptsize
    -a_{1,1}}\hspace{4pt}}\!\!\bigcirc
    \hspace{16pt}
    \bigcirc
    \!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,1}}\hspace{7pt}}\!\!\bigcirc 
    \!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -a_{2,1}}\hspace{4pt}}\!\!\bigcirc
    \hspace{17pt} \cdots \hspace{19pt}
    \bigcirc
    \!\!\xrightarrow{\hspace{7pt}{\scriptsize
    b_{K,1}}\hspace{6pt}}\!\!\bigcirc
    \!\!\xleftarrow{\hspace{3.5pt}{\scriptsize
    -a_{K,1}}\hspace{3pt}}\!\!\bigcirc
    \\[-0pt]
    & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow \hspace{19pt}
    \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow \hspace{54.5pt}
    \Big\downarrow \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{15pt}
    \Big\downarrow 
    \\[-10pt]
     & \!\bigcirc
    \!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{1,2}}\hspace{7.5pt}}\!\!\bigcirc
    \!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -a_{1,2}}\hspace{4pt}}\!\!\bigcirc
    \hspace{15.5pt}\bigcirc
    \!\!\xrightarrow{\hspace{8pt}{\scriptsize
    b_{2,2}}\hspace{7pt}}\!\!\bigcirc 
    \!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -a_{2,2}}\hspace{4pt}}\!\!\bigcirc
     \hspace{17pt} \cdots \hspace{19.5pt}
    \bigcirc
    \!\!\xrightarrow{\hspace{7pt}{\scriptsize
    b_{K,2}}\hspace{6.5pt}}\!\!\bigcirc
    \!\!\xleftarrow{\hspace{3.5pt}{\scriptsize
    -a_{K,2}}\hspace{2.5pt}}\!\!\bigcirc
  \end{align*}

* Below is an example HLS function that implements the cascade-form
  SFG with transposed-form II SoSs:
  ```c++
  // This is a Chebyshev type-II lowpass filter with M=N=6 (L=7)
  // expressed as SoSs. 
  #define K 3
  const din_t b[K][2]={
    {1.931625159186722, 0.999999999999988},
    {1.540424279915726, 1.000000000000037},
    {1.220028067120242, 0.999999999999978}
  };
  const din_t a[K][2]={
    {0.637344246881063, 0.128737933973358},
    {0.576772441445408, 0.316322235552652},
    {0.562309267750655, 0.700839985387954},
  };
  const din_t b0=0.168814352044553;

  void iir2nd(dout_t &in, dout_t &out, dout_t &w1, dout_t &w2, 
              const din_t b[2], const din_t a[2]) {
  #pragma HLS inline

    // Write to out
    out = in+w1;
    // Update register
    dout_t bx = b[0]*in;
    dout_t ay = a[0]*out;
  #pragma HLS bind_op variable=bx op=mul impl=dsp
  #pragma HLS bind_op variable=ay op=mul impl=dsp
    w1 = w2+bx-ay;
    bx = b[1]*in;
    ay = a[1]*out;
    w2 = bx-ay;
  }

  void iir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {

    static dout_t u[K+1];
    static dout_t w[K][2] = {};
  #pragma HLS array_partition variable=u type=complete
  #pragma HLS array_partition variable=w type=complete

    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      u[0] = b0*in.read();
      cascade_loop: for (int k=0; k<K; k++) {
        iir2nd(u[k], u[k+1], w[k][0], w[k][1], &b[k][0], &a[k][0]);
      }
      out.write(u[K]);
    }
  }
  ```
  Vitis HLS gives a RTL implementation of `sample_loop` with
  II=1. The latency of `sample_loop` is higher than that achieved
  using the transposed-form implementation.

## Parallel-form Implementation
* For an IIR filter ($N \geq M$) with real-valued taps and
  single-order poles, the transfer function $H(z)$ in {eq}`iir_tf` can
  be expanded into a sum of *partial fractions* :
  ```{math} 
  :label: iir_pf 
  \begin{align} 
  H(z) 
  &=
  B_0 + \sum_{k=1}^K 
  \frac{B_{k,0} + B_{k,1} z^{-1}}{1 + A_{k,1}
  z^{-1} + A_{k,2} z^{-2}} 
  \end{align} 
  ``` 
  where all the $A$ and $B$ coefficients are real-valued.

* Based on {eq}`iir_pf`, we may implement the IIR filter as parallel
  second-order IIR components in the following SFG:
  \begin{align*}
    & \!\bigcirc\!\!\xrightarrow{\hspace{32pt}{\scriptsize
    B_{0}}\hspace{31.5pt}}\!\!\bigcirc
  \\[-10pt]
     \nearrow \hspace{3pt}
     & \hspace{72pt}  \searrow
  \\[-10pt]
  \!\bigcirc\kern-6.5pt\vcenter{\tiny x}\longrightarrow
    \!\!\bigcirc\!\!\longrightarrow
    & \!\bigcirc\!\!\xrightarrow{\hspace{7pt}{\scriptsize
    B_{1,0}}\hspace{6pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{25.5pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc\kern-6.5pt\vcenter{\tiny y}
  \\[-0pt]
    \Big| \hspace{19pt}
    & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{16pt}
    \Big\downarrow 
    \hspace{18.5pt} \Big\uparrow
  \\[-10pt]
    \big| \hspace{19.0pt}
    & \!\bigcirc
    \!\!\xrightarrow{\hspace{7pt}{\scriptsize
    B_{1,1}}\hspace{6.5pt}}\!\!\bigcirc 
    \!\!\xleftarrow{\hspace{4pt}{\scriptsize
    -A_{1,1}}\hspace{4pt}}\!\!\bigcirc
    \hspace{18.2pt} \big|
  \\[-0pt]
    \Big| \hspace{19pt}
    & \hspace{33pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{16pt}
    \Big\downarrow 
    \hspace{19.8pt} \Big|
  \\[-10pt]
     \Big\downarrow \hspace{17.5pt}
     & \hspace{31.5pt}\bigcirc
    \!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -A_{1,2}}\hspace{4pt}}\!\!\bigcirc
    \hspace{17.8pt} \Big|
   \\[-0pt]
    \!\!\bigcirc\!\!\longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{7pt}{\scriptsize
    B_{2,0}}\hspace{6pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{25.5pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
  \\[-0pt]
    \Big| \hspace{19pt}
    & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{16pt}
    \Big\downarrow 
    \hspace{18.5pt} \Big\uparrow
  \\[-10pt]
    \big| \hspace{19.0pt}
    & \!\bigcirc
    \!\!\xrightarrow{\hspace{7pt}{\scriptsize
    B_{2,1}}\hspace{6.5pt}}\!\!\bigcirc 
    \!\!\xleftarrow{\hspace{4pt}{\scriptsize
    -A_{2,1}}\hspace{4pt}}\!\!\bigcirc
    \hspace{18.2pt} \big|
  \\[-0pt]
    \Big| \hspace{19pt}
    & \hspace{33pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{16pt}
    \Big\downarrow 
    \hspace{19.8pt} \Big|
  \\[-10pt]
     \Big\downarrow \hspace{17.5pt}
     & \hspace{31.5pt}\bigcirc
    \!\!\xleftarrow{\hspace{4.5pt}{\scriptsize
    -A_{2,2}}\hspace{4pt}}\!\!\bigcirc
    \hspace{17.8pt} \Big|
  \\[-0pt]
    \vdots  \hspace{19.5pt}
    & \hspace{35pt} \vdots \hspace{54.8pt} \vdots 
  \\[-0pt]
     \Big\downarrow \hspace{17.5pt}
     & \hspace{90.5pt}  \Big\uparrow
  \\[-10pt]
    \!\!\bigcirc\!\!\longrightarrow
    &
    \!\bigcirc\!\!\xrightarrow{\hspace{5pt}{\scriptsize
    B_{K,0}}\hspace{6pt}}\!\!
    \bigcirc\!\!\xrightarrow{\hspace{25.5pt}}\!\!\bigcirc
    \!\!\longrightarrow\!\!\bigcirc
  \\[-0pt]
    & \Big\downarrow  \hspace{26pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{16pt}
    \Big\downarrow 
  \\[-10pt]
    & \!\bigcirc
    \!\!\xrightarrow{\hspace{5pt}{\scriptsize
    B_{K,1}}\hspace{6.5pt}}\!\!\bigcirc 
    \!\!\xleftarrow{\hspace{4pt}{\scriptsize
    -A_{K,1}}\hspace{2pt}}\!\!\bigcirc
  \\[-0pt]
    & \hspace{33pt}
   \Big\uparrow   {\scriptsize z^{-1}} \hspace{16pt}
    \Big\downarrow 
  \\[-10pt]
     & \hspace{31.5pt}\bigcirc
    \!\!\xleftarrow{\hspace{4pt}{\scriptsize
    -A_{K,2}}\hspace{2.5pt}}\!\!\bigcirc
  \end{align*} 
  where each second-order component IIR filter is implemented in the
  transposed form II.

* Below is an example HLS function that implements the parallel-form
  SFG with transposed-form II second-order components
  ```c++
  // This is a Chebyshev type-II lowpass filter with M=N=6 (L=7)
  // expressed as a parella sum of second-order components. 
  #define K 3
  const din_t B[K][2]={
    { 0.182834372101273, -0.306910458916128},
    {-1.151247645072097, -1.125034655836861},
    {-4.777765413376408, -1.682130810244864}
  };
  const din_t A[K][2]={
    {0.562309267750655, 0.700839985387954},
    {0.576772441445408, 0.316322235552652},
    {0.637344246881063, 0.128737933973358},
  };
  const din_t B0=5.914993038391787;

  void piir2nd(din_t &in, dout_t &out, dout_t &w1, dout_t &w2,
               const din_t B[2], const din_t A[2]) {
  #pragma HLS inline

    dout_t bx = B[0]*in;
  #pragma HLS bind_op variable=bx op=mul impl=dsp
    // Write to out
    out = bx+w1;
  
    // Update register
    dout_t ay = A[0]*out;
  #pragma HLS bind_op variable=ay op=mul impl=dsp
    bx = B[1]*in;
    w1 = w2+bx-ay;
    ay = A[1]*out;
    w2 = -ay;
  }

  void iir(hls::stream<din_t> &in, hls::stream<dout_t> &out, int N) {
    dout_t w[K][2] = {};
  #pragma HLS array_partition variable=w type=complete

    sample_loop: for (int n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_N
      din_t x = in.read();
      dout_t y = B0*x;
      parallel_loop: for (int k=0; k<K; k++) {
        dout_t yy;
        piir2nd(x, yy, w[k][0], w[k][1], &B[k][0], &A[k][0]);
        y += yy;
      }
      out.write(y);
    }
  }

  ```
     Vitis HLS gives a RTL implementation of `sample_loop` with II=1. 
