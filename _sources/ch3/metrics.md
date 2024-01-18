# Performance Metrics
* In order to explain the principles that will leads to *good* HLS
  design, we need to first discuss the performance metrics that are
  commonly used to guage the "*goodness*" of a DSP kernel.

* One may consider a factory, which takes in a continuous stream of input
  materials and produces a continuous stream of output products, as an
  analogy to a DSP kernel. A common performance metric for the
  factory is the ***throughput*** which is the number of products that
  it produces per unit time. In the context of DSP, for example:
  - If the DSP kernel implements an FIR/IIR filter, the input and output are
     both streams of signal samples, and hence the throughput of the kernel
     is simply the rate of the output sample stream. 
  - If the DSP kernel implements FFT, the input is a stream of signal
    samples and the output is a stream of FFT blocks, and hence the
    throughput of the kernel is the number of FFT blocks that the
    kernel calculates per second.
  
  Using DSP terminology, the throughput measures the *steady-state*
  production rate of a DSP kernel.
  
* As in the factory analogy, the throughput achieved by a DSP kernel
  depends on the amount of resources that it consumes. More resources
  are often needed to achieve a higher throughput. We will be
  particularly interested in the amount of CLBs, DSP slices, and RAM
  consumed by a DSP kernel as well as the amount of power needed to
  sustain the operations of these PL hardware resources. A standard
  HLS development strategy is to minimize the amount of PL hardware
  resources (and power consumption) required to achieve a throughput
  goal.

* Going back again to the factory analogy, a factory typically
  produces its output products in a ***pipeline***. That is, different
  tasks in the production process are carried out by different
  stations/parts in the pipeline, and each station performs solely a
  specific task continuously in the production process. The same
  pipelining approach also applies to DSP kernels. For example, in DSP
  kernel that implements the cascade of two FIR filters, each
  component filter acts as a station in the pipeline that performs
  filtering through the cascade. On a finer level, one may also think
  of each multiply-add operation corresponding to a tap in a component
  FIR filter as a station in the pipeline that implements the FIR
  filter.

* Pertaining to the pipelining operation of a DSP kernel, we are
  primarily interested in two metrics, namely iteration latency
  and initiation interval:
  - ***Iteration latency*** of a pipeline is the duration taken from
    when the first input item enters into the pipeline to the time at
    which the pipeline produces the first output item.  Iteration
    latency specifies a *transient* characteristic of the pipeline.
  - ***Initiation interval (II)*** of a pipeline operation is the amount
    of time required to elapse between successive operations by the
    same station. Clearly, the throughput of the pipeline is
    determined by the IIs of the operations in the pipeline.
   
   It is convenient for us to express both iteration latency and II in
   terms of cycles of the clock that drives a DSP kernel.
 

