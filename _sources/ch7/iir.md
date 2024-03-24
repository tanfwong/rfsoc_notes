# IIR Filter
* Consider an $N$-order IIR filter specified by {eq}`iir` in the time
domain or {eq}`iirz` in the $z$-domain. For the sake of drawing
simpler SFGs, we assume below that the feedback order $N$ is at least
as large as the feedforward order $M$.

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
  form, we obtain the following SSFG of direct-form I for the IIR
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
    \!\bigcirc\!\!\xleftarrow{\hspace{9pt}{\scriptsize
    a_1}\hspace{9pt}}\!\!\bigcirc
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
    \!\bigcirc\!\!\xleftarrow{\hspace{8pt}{\scriptsize
    a_N}\hspace{9pt}}\!\!\bigcirc
   \end{align*}
 
  

