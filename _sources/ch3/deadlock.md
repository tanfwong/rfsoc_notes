(sec:deadlock)=
# Deadlock

* A ***deadlock*** in a data flow graph refers to the situation in
  which two or more tasks are waiting for one another to release the
  streaming buffers connecting them, resulting in no tasks can access
  any data and continue their operations indefinitely.

* Let reconsider the diamond-shaped data flow graph in
  {eq}`diamond` to see how the use of FIFOs of insufficient depth
  can cause a deadlock:
  ```{math}
  :label: deadlock
  \begin{equation}
  \boxed{I} \rightarrow A \
  \begin{array}{c} 
  \stackrel{(1)}{\nearrow} {}^{\displaystyle B} \searrow \\ 
  \stackrel{(1)}{\searrow} {}_{\displaystyle C} \nearrow 
  \end{array} 
  \ D \rightarrow \boxed{O}
  \end{equation}
  ```
  where both the FIFO connecting task $A$ to task $B$,
  $\mathcal{B}_{A,B}$, and the FIFO connecting task $A$ to task $C$,
  $\mathcal{B}_{A,C}$, have depth one. Further, assume that task A
  produces a piece of data (e.g., a signal sample) each clock cycle
  and insert pieces of data alternatively into $\mathcal{B}_{A,B}$ and
  $\mathcal{B}_{A,C}$, and that tasks $B$ and $C$ are each able to
  consume a piece of data from their respective FIFO every two clock
  cycles. Note that the insertion rate matches the consumption rate for
  $\mathcal{B}_{A,B}$ and $\mathcal{B}_{A,C}$.

* Let us denote four consecutive clock cycles by $C_1, C_2, C_3,$ and
  $C_4$.  Consider now task $A$ generates data $d_{A}[1]$ and inserts
  it into $\mathcal{B}_{A,B}$ in $C_1$. Task $B$ then reads $d_{A}[1]$
  starting at $C_2$ until the end of $C_3$. In $C_3$, task $A$
  generates another piece of data $d_{A}[2]$ and tries to insert it
  into $\mathcal{B}_{A,B}$. However, with a depth of $1$,
  $\mathcal{B}_{A,B}$ is still full in $C_3$, the write is blocked and
  task $A$ will continue to hold the write request in $C_4$ (if a
  blocking write is issued by task $A$ rather than dropping
  $d_{A}[2]$). At the same time in $C_4$, task $B$ is scheduled to
  read again from the $\mathcal{B}_{A,B}$. As a result, tasks $A$ and
  $B$ will both be waiting for each other to release the FIFO to
  continue their respective operations, creating a deadlock situation.
  The same deadlock situation also happens between tasks $A$ and $C$
  with $\mathcal{B}_{A,C}$.

* The deadlock above can be resolved by increasing the FIFO depth to
  $2$ or more for $\mathcal{B}_{A,B}$ and $\mathcal{B}_{A,C}$. With a
  larger FIFO depth, task $A$ will be able to insert $d_{A}[2]$ into
  $\mathcal{B}_{A,B}$ in $C_3$, and task $B$ will be able to continue
  its schedule of reading $d_{A}[2]$ from $\mathcal{B}_{A,B}$. The
  writing and reading sequence continues without any deadlock.

* The deadlock situation will also be prevented if PIPOs of size $1$
  are used for both $\mathcal{B}_{A,B}$ and $\mathcal{B}_{A,C}$
  instead. To see that, let $(B_1,B_2)$ be the pair of buffers of size
  $1$ used in the PIPO $\mathcal{B}_{A,B}$. As before, task $A$ writes
  $d_{A}[1]$ to $B_1$ in $C_1$, and task $B$ reads $d_{A}[1]$ from
  $B_1$ in $C_2$ and $C_3$. In $C_3$, task $A$ writes $d_{A}[2]$ to
  $B_2$, and task $B$ then reads $d_{A}[2]$ starting at $C_4$ and so
  on. The writing and reading sequence continues without any deadlock.

* Vitis HLS has the capability to detect deadlocks during C/RTL
  co-simulation.

