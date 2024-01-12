(sec:hls)=                                                                                                                       
# Introduction to HLS

* As discussed in {numref}`sec:vaad`, we will employ *high-level
synthesis (HLS)* to develop DSP kernels in the PL. 

* In the HLS design process of a DSP kernel {cite}`ug1399`, we specify
  the ***macro-architecture*** of the kernel, including:
  - the behavioral description of the DSP kernel in a high-level
    language, such as C/C++,
  - the ways that the DSP kernel interfaces with other components
     in the system, and
  - design constraints such as clock period, latency, throughput, etc.
 
 * The Vitis HLS tool then automates the ***micro-architecture***
   design of the DSP kernel, including state machine generation,
   operation scheduling, creating data path and register pipeline,
   generating control logic and interfaces for the kernel, etc, from
   the high-level C/C++ description of the DSP kernel, interface, and
   constraint specifications. The tool synthesizes the
   micro-architecture design into a RTL realization of the DSP kernel
   that can be further processed by Vitis and Vivado.
