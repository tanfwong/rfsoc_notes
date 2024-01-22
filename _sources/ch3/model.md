# Producer-Consumer Model

* In a typical DSP application scenario, a signal *source* (e.g., an ADC
  or a camera) provides a stream of samples that is fed into a DSP
  task (e.g., an FIR filter) for processing. The DSP task may
  produce an output stream of samples that is in turn fed into
  another DSP task (e.g., an FFT) for further processing, and so on,
  eventually reaching a signal *sink* (e.g., the PS host). We may
  perform each DSP task in a separate PL kernel, or package all DSP
  tasks into a single PL kernel in HLS. 

* This configuration naturally leads to the ***producer-consumer
  model*** in which a DSP task acts as a *producer* of samples and
  another DSP task downstream acts as a *consumer* of the samples
  supplied by the producer task. Some form of *buffering* is typically
  applied between the producer and consumer. The operation of a DSP
  kernel may then be described by a directed graph, called the
  ***signal/data flow graph***, representing a network of connected
  producer and consumer tasks from the input (source) of the DSP
  kernel to its output (sink).

* It is advantageous for us to develop our C/C++ design of a DSP
  kernel following this producer-consumer model because it allows the
  HLS tool to automate the optimization of **pipelining** and
  **parallelization** at the task level, as explained in the following
  sections.

## Pipelining
* Task-level pipelining is best explained by considering an
  example. Suppose the function of a DSP kernel can be decomposed into
  a sequence of three tasks, namely tasks A, B, and C in that order.
  The data flow graph of the DSP kernel is then simply
  \begin{equation*} 
  \boxed{I} \rightarrow A \rightarrow B
  \rightarrow C \rightarrow \boxed{O} 
  \end{equation*}
  where $\boxed{I}$ is the input to the kernel, $\boxed{O}$ is the
  kernel output, and each directed edge $\rightarrow$ shows the
  direction of the producer-consumer relation and buffering between
  two connected tasks.  The IIs of the three tasks are $\tau_A$,
  $\tau_B$, and $\tau_C$ clock cycles, respectively. For illustration
  below, let us assume $\tau_B > \tau_A > \tau_C$.

* If the producer-consumer model is not followed when developing the
  DSP kernel that task A has to wait for both tasks B and C to complete their
  respective operations on the output unit itself produces before it
  starts to work on another input unit, then both the iteration
  latency and II of the DSP kernel will be $\tau_A + \tau_B + \tau_C$,
  and the throughput cannot be higher than $\frac{1}{\tau_A + \tau_B +
  \tau_C}$ output units per clock cycle.

* Under the producer-consumer model, with sufficient buffering between
  the tasks, instead of sitting idle waiting for the other two downstream
  tasks to complete their processing on its output, task A can
  start processing the next input unit in a *pipelining* fashion as
  shown in the figure below:
  ```{figure} ../figs/pipeline.jpg
  ---
  name: pipeline
  alt: Task-level pipelining
  width: 800px
  align: center
  ---
  Timing diagram showing task-level pipelining in a DSP kernel
  constructed under the producer-consumer model with tasks A, B, and
  C of different IIs.
  ```
  With this form of task-level pipelining, it is easy to check that
  while the iteration latency of the DSP kernel is still $\tau_A +
  \tau_B + \tau_C$, the II drops down to $\tau_B$ and the throughput
  increases to $\frac{1}{\tau_B}$.

* We see from the example above that task-level pipelining can
  decrease the II and increase the throughput of a DSP kernel while
  not requiring additional PL resources, except perhaps for additional
  buffering and more complex control logic. However, *task-level
  pipelining does require the IIs of all tasks to be known at the
  build time for the HLS tool to construct the pipelining schedule*.

## Parallelization
* When two DSP tasks are *fully independent*, i.e., their operations
  do not depend on one another, they do not communicate, and they do
  not access any shared memory or computational resource, they can be
  executed in parallel; thus reducing the II and increasing the
  throughput of the DSP kernel.

* A piece of C/C++ code written in the producer-consumer model helps
  the HLS tool to identify opportunities for task-level
  parallelization. For example, the following data flow graph shows
  that tasks B and C are fully independent and hence can be executed
  in parallel: 
  \begin{equation*} 
  \boxed{I} \rightarrow A \
  \begin{array}{c} 
  \nearrow {}^{\displaystyle B} \searrow \\ 
  \searrow {}_{\displaystyle C} \nearrow 
  \end{array} 
  \ D \rightarrow \boxed{O}
  \end{equation*}

* With task-level parallelization, independent tasks are executed
  simultaneously on different PL resources. Hence, unlike pipelining, 
  parallelization does require more PL resources. 
