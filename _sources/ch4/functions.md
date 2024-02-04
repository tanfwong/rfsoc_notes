# Functions

* All functions and sub-functions are synthesized into separate
  modules in the RTL specification of a DSP kernel, unless they are
  inlined (see below).
  
## Top-level Function
* Each DSP kernel must have a top-level function, which is identified
  by a configuration parameter in Vitis HLS (see the instructions of
  Lab 2). 

* The top-level function is synthesized into the top-level module in
  the RTL specification.

* Data access to the kernel from outside (the host, the Vitis platform, other
  kernels, and the test bench) must go through an argument of the
  top-level function. The arguments of the top-level function are
  synthesized into interfaces to external hardware components. 

* Under the VAAD flow, the default interfaces synthesized for
  different argument types are listed in the table below:
  ```{list-table} Default access paradigms for different types of top-level arguments under the VAAD flow
  :name: top_func_args
  :header-rows: 1

  * - Argument type
    - Data access paradigm
    - Interface protocol

  * - Scalar (pass by value)
    - Register
    - AXI4 Lite (`s_axilite`)

  * - Pointer to scalar
    - Register
    - AXI4 Lite (`s_axilite`)

  * - Array
    - Memory
    - AXI4 Memory Mapped (`m_axi`)
  
  * - Pointer to array
    - Memory
    - AXI4 Memory Mapped (`m_axi`)
  
  * - Reference
    - Register
    - AXI4 Lite (`s_axilite`)

  * - `hls::stream`
    - Stream
    - AXI4 Stream (`axis`) 

  ```
