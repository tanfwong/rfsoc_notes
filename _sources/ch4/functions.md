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

## Function Inlining 
* Inlining a function is a standard C++ optimization technique that
  dissolves the function logic into the calling function. In HLS,
  after a function is inlined, it will not be synthesized into a
  separate RTL module; instead the function logic is embedded into
  that of the calling function. 

* Inlining a function could improve the usage efficiency of PL
  resources by sharing the logic components in the function with the
  calling function as well as by optimizing the control logic of the
  calling function. It may potentially reduce the II of the calling
  function.  However, since an inlined function is no longer
  synthesized as a separate RTL module, it can not be shared or
  reused. Thus, multiple calls to the inlined function may consume
  more PL resources due to replication. Often, we may need to use Vitis
  HLS to synthesize different designs with and without inlining to
  determine if any advantage in the PL resource utilization can be
  achieved with function inlining.

* Function inlining is automatically performed by Vitis HLS for small
  functions. We can involve or turn off function inlining in Vitis HLS
  using [`#pragma HLS
  inline`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-inline)
  as shown in the example below:
  ```c++
  void foo (int &p, int &q) {
  #pragma HLS inline off
    q += p;
  }
    
  void sub(int &p, int &q) {
  #pragma HLS inline
    int p1 = p + 1;
    foo(p1, q); //foo_3
  }
  
  void diamond(int &a, int &b, int &c, int &d) {
  #pragma HLS allocation function instances=foo limit=1 
    foo(a, b); //foo_1
    foo(c, d); //foo_2
    sub(b, d);
  }
  ```
  - The instance `sub(b, d)` of the function `sub()` is inlined into the
    top-level function `top()` as directed by the inline pragma in the
    body of `sub()`. 
  - By default, inlining a function applies only to the level of the
    function body; hence inlining of `sub()` does not recursively
    apply to the call to `foo()` inside `sub()`. In this example,
    inlining of `foo()` is explicitly turned off. Removing the line
    `#pragma HLS inline off` from `foo()` will cause Vitis HLS to
    automatically inline `foo()` due to its simplicity.
  - The pragma [`#pragma HLS
    allocation`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-allocation)
    in `top()` limits that only a single instance of `foo()` will be
    synthesized. Without it, two instances of `foo()` will be
    synthesized by Vitis HLS for task-level parallelization. This
    pragma provides us finer control on the tradeoff between PL
    resource utilization and iteration latency of the kernel.
  
