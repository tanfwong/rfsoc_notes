# OpenCL Platform

* [OpenCL](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/README.md),
  which stands for Open Computing Language, is an industrial standard
  to support cross-platform parallel programming in computing systems
  with hetergeneous compute resources, such as CPUs, GPUs, FPGAs, dsps,
  and TPUs

* The OpenCL *platform model* describes how the compute resources in a
  system are topologically connected. The typical OpenCL platform
  model is shown in the following figure:
  ```{figure} ../figs/opencl_platform.jpg
  ---
  name: opencl_platform
  alt: OpenCL Platform Model
  width: 800px
  align: center
  ---
  OpenCL Platform Model (figure taken from
  [here](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/images/platform_model.jpg))
  ```
  - A *host* (usually a CPU) is connected to one or more *compute
    devices* (CPUs, GPUs, FPGAs, dsps, TPUs), each of which is in a
    collection of one or more *compute units*. Each compute unit may
    in turn consists of one or more *processing elements*.
  - The host controls the compute devices whose compute units may
    perform computing tasks in parallel.
  - Correspondingly in our case, we may regard the XCZU48DR RFSoC
    device as our *platform*, in which the ARM Cortex-A53 APU is the
    *host* and the PL is a *compute device*. The DSP kernels in the PL
    are *compute units*, each of which is composed of the tasks in the
    data flow graph that defines the kernel serving as *processing
    elements*.

* OpenCL provides a programming framework for us to develop programs
  that run on the host as well as the compute devices. In particular,
  the OpenCL framework contains two layers of APIs:
  - The *platform-layer APIs* support an application program that runs
    on the host to discover the platform architecture, and to set up,
    configure, and initialize the compute devices in the platform.
  - The *runtime APIs* support the development of *kernel programs*
    that run on the compute devices. Some runtime APIs also run on the
    host to transfer kernel programs and data to the compute units in
    the compute devices, execute the kernel programs to perform
    computing tasks in the compute units, and transfer results back
    from the compute devices back to the host.

* As discussed in {numref}`sec:vaad`, we will use Vitis HLS to develop
  kernel programs in C/C++, we will not use the runtime APIs that
  support that purpose. However, we will use both the platform-layer
  and runtime APIs in the host program to set up the OpenCL platform,
  control the DSP kernels, and execute the DSP computing task in the
  kernels.
 
