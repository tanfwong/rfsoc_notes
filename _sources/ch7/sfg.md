# Block Diagram & Signal Flow Graph

## Block Diagram
* All FIR and IIR filters can be constructed using the following three
  basic components:
  1. **Two-input adder**:
    \begin{align*}
    & x_2[n] \\
    &\,\big\downarrow \\[-5pt]
    x_1[n] \longrightarrow & \!\bigoplus \!\!\longrightarrow  x_1[n] + x_2[n]
    \end{align*}

  2. **Scalar multiplier**:
    \begin{align*}
    & \,a \\
    &\,\big\downarrow \\[-5pt]
    x[n] \longrightarrow & \!\bigotimes \!\!\longrightarrow  ax[n]
    \end{align*}

  3. **Unit delay**:
    \begin{align*}
    x[n] \longrightarrow & \!\boxed{z^{-1}_{}} \!\!\longrightarrow  x[n-1]
    \end{align*}

* The construction of an FIR/IIR filter using these three basic
  component is often expressed in the form of a block diagram.
  - *Example 1*:
    ```{figure} ../figs/fir_ex_bd.jpg
    ---
    name: fir_ex_bd
    alt: Block diagram of FIR filter
    width: 600px
    align: center
    ---
    Block diagram of FIR filter
    $y[n] = 2x[n] - 0.5x[n-1] + 0.2x[n-2]$
    ```
  - *Example 2*:
    ```{figure} ../figs/iir_ex_bd.jpg
    ---
    name: iir_ex_bd
    alt: Block diagram of IIR filter
    width: 600px
    align: center
    ---
    Block diagram of IIR filter
    $y[n] = 0.9y[n-1] - 0.5y[n-2] + x[n]$
    ```

## Signal Flow Graph
* The block diagram of an FIR/IIR filter can be represented by a
  *signal flow graph (SFG)*. It is often more convenient to draw the SFG
  of a higher-order FIR/IIR filter than the block diagram
  representation.

* The SFG of a filter is a *(edge-)weighted directed graph* $G=(V,E)$ consisting of:
  - a set $V$ of vertices that are signals generated in the block
    diagram of the filter,
  - a set $E$ of edges, each of which is an ordered pair of vertices
    showing the direction of the signal flow,
  - at least one *source* vertex in $V$ with no incident edges, and 
  - at least one *sink* vertex in $V$ with no emanating edges.
  
  In addition, the vertices and the edges must satisfy the following
  conditions:
  - Each edge is associated with a *gain factor (weight)* that is
    either a scalar constant or $z^{-1}$. The gain factor is applied
    to the first signal (vertex) to obtain the second signal (vertex)
    of the edge. If no gain factor is specified, the default value of
    $1$ applies.  For example, the edge $(w_1,w_2)$ with a gain factor
    $z^{-1}$ means $W_2(z) = z^{-1} W_1(z)$ in the $z$-domain, or
    equivalently $w_2[n] = w_1[n-1]$ in the time domain. For
    convenience, we may write the edge as $(w_1, w_2; {z^{-1})}$ and
    draw it as $\bigcirc\kern-7.5pt\vcenter{\tiny w_1}
    \!\xrightarrow{~z^{-1}~}\!\!\bigcirc\kern-7.5pt\vcenter{\tiny
    w_2}~$.

  - The signal associated with a vertex is the "output" signal in the
    sense that it is the sum of the incident signals to the
    vertex. For example, consider that the two unit-weighted edges
    $(w_1,w_3)$ and $(w_2,w_3)$, and that the signal $w_3$ has only
    $w_1$ and $w_2$ as its incident signals. Pictorially, 
    ```{math}
    \begin{align*}
    & \!\bigcirc\kern-7.5pt\vcenter{\tiny w_1}\\[-5pt]
    & \big\downarrow \\[-5pt]
    \bigcirc\kern-7.5pt\vcenter{\tiny w_2}\!\!\longrightarrow &
    \!\bigcirc\kern-7.5pt\vcenter{\tiny w_3}\!\!\longrightarrow 
    \end{align*}
    ```
    This means $w_3[n] =
    w_1[n]+w_2[n]$ in the time domain, or equivalently $W_3(z) =
    W_1(z)+W_2(z)$ in the $z$-domain.


 

