# Filter Implementation

* We will discuss in this section implementation of finite impulse
  response (FIR) and infinite impulse response (IIR) filters in HLS.

* Recall that both FIR and IIR filters can be defined by linear
  I/O difference equations in the time domain:
  - **FIR** filter of order $M$:
    ```{math}
    :label: fir
    \begin{align} 
    y[n] = \sum_{k=0}^M b_k x[n-k] & \hspace{90pt} & (b_0 \neq 0) 
    \end{align}
    ```
  - **IIR** filter of *feedforward* order $M$ and *feedback* order $N$
    (or simply of order $N$):
    ```{math}
    :label: iir
    \begin{align} 
    \sum_{k=0}^N a_k y[n-k] = \sum_{k=0}^M b_k x[n-k] & \hspace{50pt} &
    (a_0 =1) 
    \end{align}
    ```
    where $x[n]$ and $y[n]$ denote the input and output signals,
    respectively.

* Equivalently, in the $z$-domain, the above difference equations are
  transformed to the following algebraic equations:
  - **FIR** filter of order $M$:
    ```{math}
    :label: firz
    \begin{align} 
    Y(z) = \sum_{k=0}^M b_k z^k X(Z) & \hspace{80pt} & (b_0 \neq 0) 
    \end{align}
    ```
  - **IIR** filter of *feedforward* order $M$ and *feedback* order $N$
    (or simply of order $N$):
    ```{math}
    :label: iirz
    \begin{align} 
    \sum_{k=0}^N a_k z^k Y(z) = \sum_{k=0}^M b_k z^k X(z) & \hspace{50pt} &
    (a_0 =1) 
    \end{align}
    ```
    where $x[n] \stackrel{z}{\longleftrightarrow} X(z)$ and $y[n]
    \stackrel{z}{\longleftrightarrow} Y(z)$.
