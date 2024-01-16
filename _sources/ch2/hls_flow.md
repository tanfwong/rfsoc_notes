# Vitis HLS Development Flow

* As discussed in {numref}`sec:hls`, we are responsible for the
  macro-architecture design of a DSP kernel in the HLS development
  process. Vitis HLS will then automate the micro-architecture
  design of the DSP kernel and synthesize the RTL implementation
  following the steps described in {numref}`sec:hls_tasks`. 

* The standard sequence of steps to use Vitis HLS for DSP kernel
  development in the VAAD flow are listed blow:
  1. ***C/C++ Kernel Coding***: Develop the C/C++ specification of the
     DSP kernel, and specify the external interfaces, design
     constraints, and directives.
  2. ***C-Simulation***: Verify the functionality of the C/C++ kernel
     code with the C/C++ test bench.
  3. ***Code Analysis***: Analyze the performance, parallelism, and
     legality of the C/C++ kernel code.
  4. ***C-Synthesis***: Generate the RTL specification for the given
     clock speed and input constraints using the v++ compiler.
  5. ***C/RTL Co-Simulation***: Verify the RTL code generated using
     the C/C++ test bench, and review the HLS synthesis and
     implementation timing reports. If necessary, re-run previous
     steps until performance goals are met.
  6. ***Package***: Generate the DSP kernel object (`.xo`) for use in
     the VAAD flow.

 ```{figure} ../figs/hls_steps.png
 ---
 name: HLS steps
 alt: Steps in the HLS development flow
 width: 600px
 align: center
 ---
 Steps in the Vitis HLS development flow 
 (image taken from {cite}`ug1399`)
 ```

* In the next few sections, we will briefly describe some basic design
  principles, considerations, and techniques in writing optimized
  C/C++ code for DSP development using HLS.
