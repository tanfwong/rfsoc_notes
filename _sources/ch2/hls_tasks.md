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
  for the DSP kernel includes the data path, control logic, and memory,
  AXI4, and other interfaces. The main functionality of the DSP kernel
  is implemented in the *data path*, which consists of:
   - storage elements, e.g., registers and memories, 
   - functional units, e.g., multipliers and arithmetic logic units, and 
   - interconnect elements, e.g., multiplexers, tri-state drivers, buses. 
 

* The flowchart below shows the sequence of tasks that the HLS tool
  needs to perform in order to produce the RTL specification of a DSP
  kernel from the high-level C/C++ specification:
  ```{figure} ../figs/hls_tasks.png
  ---
  name: hls_tasks
  alt: HLS tasks
  width: 600px
  align: center
  ---
  Sequence of tasks performed by the HLS tool
  (image taken from {cite}`ug1399`)
  ```
