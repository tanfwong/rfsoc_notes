(sec:hls)=                                                                                                                       
# Introduction to HLS

* As discussed in {numref}`sec:vaad`, we will employ *high-level
synthesis (HLS)* to develop DSP kernels in the PL. 

* In the HLS design process of a DSP kernel, we specify the
  ***macro-architecture*** of the kernel, including:
  - the behavioral description of the DSP kernel in a high-level
    language, such as C/C++,
  - the ways that the DSP kernel interfaces with other components
     in the system, and
  - design constraints such as clock period, latency, and throughput.
 
