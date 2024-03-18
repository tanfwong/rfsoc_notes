# Signal Flow Graph

* All FIR and IIR filters can be constructed using the following three
  basic components:
  1. **Two-input adder**:
    \begin{align*}
    & x_2[n] \\
    &\,\big\downarrow \\
    x_1[n] \longrightarrow & \!\bigoplus \!\!\longrightarrow  x_1[n] + x_2[n]
    \end{align*}

  2. **Scalar multiplier**:
    \begin{align*}
    & \,a \\
    &\,\big\downarrow \\
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

    

 

