# OpenCL

* [OpenCL](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/README.md),
  which stands for Open Computing Language, is an industrial standard
  to support cross-platform parallel programming in computing systems
  with hetergeneous compute resources, such as CPUs, GPUs, FPGAs, dsps,
  and TPUs

## Platform Model
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
  kernel programs in C/C++. Thus, we will not use the runtime APIs that
  support that purpose. However, we will use both the platform-layer
  and runtime APIs in the host program to set up the OpenCL platform,
  control the DSP kernels, and execute the DSP computing task in the
  kernels.
  
## Memory Model
* The OpenCL *memory model* describes how different memory resources
  in the system are connected to the various components in the OpenCl
  platform. The typial OpenCL memory model is shown in the following
  figure:
  ```{figure} ../figs/opencl_memory.jpg
  ---
  name: opencl_memory
  alt: OpenCL Memory Model
  width: 800px
  align: center
  ---
  OpenCL Memory Model (figure taken from
  [here](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/images/memory_model.jpg))
  ```
  - **Host memory**: accessible only to the host
  - **Global memory**: accessible to all compute units in a compute
    device
  - **Local memory**: available only within in a compute unit
  - **Private memory**: available only to a single processing element

* Again, mapping this memory model back to our case (see {numref}`sec:hardware`):
  - The DDR4 bank connected to the PS is the host memory.
  - The DDR4 back connected to the PL is the global memory.
  - Local memory includes all memory resources within a DSP kernel
    generated for global variables, variables in the top-level
    function,  and streaming buffers.
  - Private memory includes all memory resources generated within a
    task function.

* Memory management in OpenCL is explicit. That is, the host
  application program and kernel programs must explicitly moves data
  between the different types of memory types above. In particular,
  the host program is expected to employ the OpenCL runtime APIs to
  move data between the host and global memory to implement data
  transfer between the host and a compute device.


 
