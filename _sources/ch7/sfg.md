# Signal Flow Graph

* All FIR and IIR filters can be constructed using the following three
  basic components:
  1. **Two-input adder**:
    \begin{align*}
    & x_2[n] \\
    &\downarrow \\
    x_1[n] \longrightarrow & \!\bigoplus \!\!\longrightarrow  x_1[n] + x_2[n]
    \end{align*}

  2. **Scalar multiplier**:
    \begin{align*}
    & \,a \\
    &\downarrow \\
    x[n] \longrightarrow & \!\bigotimes \!\!\longrightarrow  ax[n]
    \end{align*}

  3. **Unit delay**:
    \begin{align*}
    x[n] \longrightarrow & \!\boxed{z^{-1}_{}} \!\!\longrightarrow  x[n-1]
    \end{align*}

