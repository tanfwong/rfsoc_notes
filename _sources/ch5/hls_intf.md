# HLS Kernel Interface

* Under the VAAD flow, Vitis HLS automatically defines the interface
  of a DSP kernel so that the PS host can pass control commands to the
  kernel, and data can be exchanged between the kernel and the PS
  host, as well as with other kernels and externel hardware
  components. All discussions about the kernel interface in this
  section primarily pertain to VAAD kernels.

* The interface contains the following 2 elements:
  - a set of *data channels* for exchange of data between the kernel
    and external components, and a port protocol for each data channel
    to control the flow of data through that channel,
  - a *control channel* and the accompanying block control protocol
    `ap_ctrl_chain` by which the PS host controls the execution of the
    kernel.

* Vitis HLS automatically generates the following default hardware
  interface ports (and the corresponding adapters) in the kernel to
  handle the flow of data and control information through the data and
  control channels:
  - a clock port `ap_clk`, a reset port `ap_rst_n`, and an interrupt
     port `interrupt`for kernel control,
  - an AXI4 Lite (`s_axilite`) interface, called `s_axi_control`, for
    implementation of the block control protocol `ap_ctrl_chain`,
    management of address offset information for AXI4 memory
    mapped interfaces, and register-based data access,
  - AXI4 memory mapped (`m_axi`) interfaces for memory-based data
    access if needed, and
  - AXI4 stream (`axis`) interfaces for stream-based (sequential) data
    access if needed.
 
* Vitis HLS will always generate the clock, reset, interrupt ports and
   the `s_axi_control` interface. No other `s_axilite` interface is
   allowed under the VAAD kernel design.
 
* The data channels are constructed based on the arguments of the
  kernel's top-level function as discussed in
  {numref}`sec:top-level-function`. Hence, data access from outside
  the kernel must be defined through the top-level function's
  arguments. The default data access paradigm and interface protocol
  for basic argument types are specified in {numref}`top_func_args`.
  The default data access paradigm and interface protocol for some of
  these basic types may be overridden by using [`#pragma HLS
  interface`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-interface).
  For example, the following C++ code snippet
  ```c++
  void top(int in[N], int out[N]) {
  #pragma HLS interface axis port=in
  #pragma HLS interface axis port=out
    ...
  }
  ```
  specifies access to the arrays `in[N]` and `out[N]` be implemented
  using the `axis` protocol for stream-based access instead of the
  default `m_axi` protocol for memory-based access. 

* Vitis HLS however does not automatically determine default interface
  protocols for the member elements of composite arguments of the top-level
  function. We must explicitly define a suitable interface protocol
  for each member of a composite argument using the interface pragma. 

* All register-based data channels are bundled into the
  `s_axi_control` interface.  All memory-based channels will be  bundled
  into a default `m_axi` interface, called `m_axi_gmem`. Similarly,
  all stream-based channels will be bundled into a single `axis`
  interface by default. However, we may instruct Vitis HLS to generate
  more than one `m_axi` (`axis`) interface and assign memory-based
      (stream-based) channels to different `m_axi` (`axis`) interfaces
  by using the `bundle=` option of the interface pragma. 

* As Vitis HLS abstracts away the detailed hardware implementation of
  the kernel interface, we will skip over the details of the AXI4
  protocols `s_axilite`, `m_axi`, and `axis` here. Detailed
  specification of the AXI4 protocols can be found in {cite}`axi4`,
  and details about the adapter design for these protocols can be
  found in {cite}`ug1037`. Instead, we will focus on HLS programming
  techniques to allow efficient data access between a DSP kernel and
  external hardware components, in particular, the global memory. 
