# HLS Kernel Interface

* Under the VAAD flow, Vitis HLS automatically defines the interface
  of a DSP kernel so that the PS host can pass control commands to the
  kernel, and data can be exchanged between the kernel and the PS
  host, as well as with other kernels and externel hardware
  components.

* The interface contains the following 3 elements:
  - a set of *data channels* for exchange of data between the kernel and
    external components,
  - a port protocol for each data channel to control the flow of data
    through that channel, and
  - an execution control scheme (protocol) by which the PS host 
    controls the execution of the kernel.
  
