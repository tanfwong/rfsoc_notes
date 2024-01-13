# HLS Tasks

* A high-level C/C++ specification of a DSP kernel is
  ***untimed***. This means that the C/C++ specification describes
  only the logical operations in a sequential manner. No description is
  provided about the synchronous timing of the operations in the
  sequence with respect to any clock. Indeed, if the C/C++
  specification were built to run on a CPU, the time it would take to
  complete an operation as well as the transition time from one
  operation to the next would both be stochastic in general. 

* A RTL specification of a DSP kernel is, on the other hand,
  ***timed*** in the sense that all operations in the specification
  are synchronous to a clock. Thus, it is the job of the HLS tool to
  construct a timed RTL specification of the DSP kernel from the
  untimed C/C++ specification. In addition, the HLS tool needs to
  construct a digital architecture to implement the logical operations
  described in the C/C++ specification. A typical digital architecture
  for the DSP kernel includes:
  - data path 
  - control logic
  - memory, AXI4, and other interfaces. 
 
* The main functionality of the DSP kernel
  is implemented in the *data path*, which consists of:
   - storage elements, e.g., registers and memories, 
   - functional units, e.g., multipliers and arithmetic logic units, and 
   - interconnect elements, e.g., multiplexers, tri-state drivers, buses. 
 

* The list below shows the sequence of tasks that the HLS tool
  needs to perform in order to produce the RTL specification of a DSP
  kernel from the high-level C/C++ specification:

  1. ***Compilation***: The HLS tool compiles the C/C++ code that
     specifies the DSP kernel. Code optimizations are also performed
     to try to meet the design constraints. This step is equivalent to
     the compilation step in a typical C/C++ compiler, but with
     perhaps a different set of optimization objectives.

  2. ***Scheduling***: The HLS tool determines which operations should
     occur in each clock cycle and the amount of PL resources needed to
     support the operations in that clock cycle.

  3. ***Binding***: The HLS tool assigns specific PL resources to
     implement each scheduled operation as well as memories and
     registers to variables in the C/C++ code. Resource sharing is
     also performed so that the same sets of resources can be re-used
     by operations scheduled to different clock cycles. RTL
     implementations of the scheduled operations are produced in this
     step.

  4. ***Control logic extraction***: The HLS tool creates a finite state
     machine (FSM) that is responsible for sequencing the scheduled
     operations.

  5. ***Interface generation***: The HLS generates external interfaces
     for the DSP kernel.

  6. ***RTL generation***: The HLS synthesizes the RTL specification of
     the complete DSP kernel.

* The following two illustrative examples, taken directly from
  {cite}`ug1399`, help to explain the HLS tasks listed above.


## Scheduling and Binding Example
* Consider the following C/C++ function that specifies a simple DSP
  kernel: 
   ```c++ 
   int foo(char x, char a, char b, char c) { 
     char y; 
     y = x*a+b+c; 
     return y; 
   } 
   ``` 
  where the `char` variable `x` and `int` variable `y` respectively are
  the input and output of the kernel and the `char` variables `a`, `b`,
  and `c` are kernel parameters.

* The figure below shows the assignments produced by the HLS tool in
  the scheduling and binding tasks:
  ```{figure} ../figs/hls2-3.png
  ---
  name: HLS tasks 2 & 3
  alt: HLS scheduling and binding tasks
  width: 800px
  align: center
  ---
  Example showing the assigments in the HLS scheduling and binding
  tasks (image taken from {cite}`ug1399`)
  ```
