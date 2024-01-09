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
  1. Follow the *Vitis Application Accelerated Development Flow (VAADF)*. 
  2. Use HLS to develop DSP kernels under the Vitis Application
     Accelerated Development Flow.  We will discuss more about the HLS
     design process in {numref}`sec:hls`.

## Vitis Application Accelerated Development Flow
* The VAADF {cite}`ug1393` is essentially a design process that allows
  developers to focus on developing the core functionalities of the PL
  kernels while the interface development is all automated by Vitis.

* It contains:
  - a *Vitis extensible platform* which is composed of 
    - a *domain* with the necessary software to run Linux on the PS host
    - a base PL hardware block with a predefined configuration of AXI4
      interfaces for PL kernel control by the PS host and for data
      transfer with between the PS host and the PL kernels via global
      memory, clocks, and interrupted signals
  - a predefined configuration of AXI4 interfaces for kernel control
      and data transfer, clock input, and interrupt signals to which
      each PL kernel must conform
  - the [Xilinx Runtime
    (XRT)](https://xilinx.github.io/XRT/2023.2/html/index.html)
    library which contains Linux drivers and APIs to support PL kernel
    control and data transfer in host application programming.

* In addition, Vitis HLS can be employed to automatically synthesize
  interfaces that conform to the predefined configuration required by
  VAADF for a PL kernel.

* The tradeoffs for adopting VAADF are potential losses in
  flexibility of the implementation architecture and in PL utilization
  efficiency. Nonetheless, these potential losses are rather
  negligible for us since we will not be interested in developing
  interfaces to other peripherals, and the adoption of VAADF allows us
  to focus on the "DSP stuff" in our development. 


