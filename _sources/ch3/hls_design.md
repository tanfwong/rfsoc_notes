(sec:hls_design)=
# HLS Design Basics

* As discussed in {numref}`sec:hls_steps`, we start the HLS
  development process of a DSP kernel by writing C/C++ code to specify
  the DSP functionality of the kernel. The HLS tool will then
  translate our untimed C/C++ specification to a timed RTL one, going
  through the list of tasks described in {numref}`sec:hls_tasks`.

* While this first step of HLS development is similar to writing a
  piece of software code to run on a classical general-purpose CPU,
  there are some significant differences in writing C/C++ code for HLS
  kernel development because the goal of the HLS development is to
  produce a hardware specification that can efficiently utilize the
  hardware resources in the PL in order to meet our performance
  target. As the hardware architecture, resources, and contraints of
  the PL are very different from their counterparts in a
  general-purpose CPU, standard good coding practices in classical
  C/C++ development may not be applicable to writing C/C++ code for
  HLS development.
  
* The two main considerations in HLS kernel code development are to:
  1. Write C/C++ code that is *synthesizable*. Due to the limitations
       on the PL hardware architecture and resources, the HLS cannot
       synthesize some standard C/C++ constructs, such as dynamic
       memory allocation. We should avoid writing HLS code with
       non-synthesizable C/C++ constructs.
 
   2. Follow good HLS coding practices so that the HLS tool can
      produce efficient RTL implementation of the DSP kernel that
      meets performance targets.

* In order to understand what constitue good HLS programming
  practices, we will discuss some basic HLS design principles in the
  following sections.
  
