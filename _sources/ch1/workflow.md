# Vitis Application Accelerated Development

* As discussed in {numref}`sec:vitis`, we have to program the
  different compute components and construct interfaces (both hardware
  and software) between them in the typical embedded system
  development process. In particular, the development of the AXI4
  interfaces between the DSP kernels and the DMA interface with the
  global memory can be rather laborious. While the implementation of
  these interfaces may serve as good design exercises in a digital
  design class, it is not exactly the focus of what we want to do
  here, namely DSP kernel implementation.

* In order to simplify our development process, we will employ the
  following approach:
  1. Follow the *Vitis Application Accelerated Development Flow*. 
  2. Use HLS to develop DSP kernels under the Vitis Application Accelerated Development Flow.  We will discuss more about the HLS
     design process in {numref}`sec:hls`.

## Vitis Application Accelerated Development Flow
