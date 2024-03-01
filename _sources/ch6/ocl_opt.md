# OpenCL Optimization

* In many practical scenarios, the DSP kernel may process a long
  sequence of input samples (data) and produce a long sequence of
  output samples (data). As the PL has limited memory resources, the
  long sequence of input samples will need to be broken down into
  shorter blocks. Each shorter input block is passed from the host to
  the DSP kernel for processing and then the output block is passed
  back from the kernel to the host. This process repeats block by
  block until all the blocks generated from the original long input
  sequence are processed.

* This block processing approach necessitates the step sequence of
  transferring input data from the host memory to the global memory,
  executing the kernel to process the data, and transferring the
  output data from the global memory back to the host memory (steps
  7-9) be repeatedly applied from block to block.

* Since the latency of data transfer and time overhead of launching
  the DSP kernel are significantly, sequentially repeating the step
  sequence from block to block may significantly reduce the processing
  throughput. 

* In order to overcome this inefficiency, we need to optimize the OpenCL
  host code to support pipelining the steps of data transfer and
  kernel execution as well as parallelization with multiple compute
  units in the PL.
