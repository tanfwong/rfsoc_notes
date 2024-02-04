# HLS Programming

* As discussed in {numref}`sec:pro-con` and {numref}`sec:dataflow`,
 the HLS development of a DSP kernel amounts to constructing a data
 flow graph with connected task functions.

* Further understanding of what constitute good HLS coding practices
  requires us to learn about the default behaviors of Vitis HLS when
  synthesizing common C/C++ constructs, such as functions, loops,
  arrays, and various data types, in RTL code, as well as, how to use
  HLS directives and pragmas to instruct Vitis HLS to perform task-
  and instruction-level optimization in the synthesis process
  following the way that we want it to.

* This more detailed knowledge of HLS programming techniques would be
  helpful for us in the development of C/C++ specifications of DSP
  tasks functions that allow Vitis HLS to generate an efficient
  hardware design meeting specific performance goals.

* In this section, we summarize some standard HLS programming
  considerations and techniques from {cite}`ug1399` that are useful
  for developing DSP kernels.
