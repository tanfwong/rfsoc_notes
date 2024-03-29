# OpenCL Basics

[OpenCL](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/README.md),
which stands for Open Computing Language, is an industrial standard to
support cross-platform parallel programming in computing systems with
hetergeneous compute resources, such as CPUs, GPUs, FPGAs, dsps, and
TPUs

## Platform Model
* The OpenCL *platform model* describes how the compute resources in a
  system are topologically connected. The OpenCL platform
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
  the OpenCL framework contains three layers of APIs:
  - The *platform-layer APIs* support an application program that runs
    on the host to discover the platform architecture, and to set up,
    configure, and initialize the compute devices in the platform.
  - The *kernel compiler* generates *kernel programs* that run on the
    compute devices.
  - The *runtime APIs* support the application program that runs on
    the host to transfer kernel programs and data to the compute units
    in the compute devices, execute the kernel programs to perform
    computing tasks in the compute units, and transfer results back
    from the compute devices back to the host.

* As discussed in {numref}`sec:vaad`, we will use Vitis HLS to develop
  kernel programs in C/C++. Thus, we will not use the kernel compiler
  that supports that purpose. However, we will use both the
  platform-layer and runtime APIs in the host program to set up the
  OpenCL platform, control the DSP kernels, and execute the DSP
  computing task in the kernels.
  
## Memory Model
* The OpenCL *memory model* describes how different memory resources
  in the system are connected to the various components in the OpenCl
  platform. The OpenCL memory model is shown in the following
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

## Execution Model
* The OpenCL *execution model* describes how computing tasks are
  executed in the different compute devices in the OpenCl
  platform. The typial OpenCL execution model is shown in the following
  figure:
  ```{figure} ../figs/opencl_execution.jpg
  ---
  name: opencl_execution
  alt: OpenCL Execution Model
  width: 800px
  align: center
  ---
  OpenCL Execution Model (figure taken from
  [here](https://github.com/KhronosGroup/OpenCL-Guide/blob/main/images/executing_programs.jpg))
  ```
* To execute computing tasks in a platform, we must first create a
  **context**, which includes a set of compute devices, the memory
  objects accessible to those compute devices, and one or more command
  queues that are used to schedule execution of kernel programs and
  operations on memory objects. In essence, the context is an
  environment within which the kernel programs execute and memory
  management and synchronization are performed.

* A **kernel program** executes on a compute device. This corresponds
  to the kernel code of our DSP kernel. Multiple instances of the same
  kernel program can be implemented in the PL forming individual
  compute units. We will slightly carelessly use the term **kernel**
  to refer to both the kernel code and the compute unit that is
  implemented in the PL. An OpenCL **program** is a collection of
  kernels.  A **memory object** is a handle to a region of the global
  memory.
  
* A **command queue** holds commands that will be executed on a
  specific compute device.  The host places commands into the command queue to
  schedule their execution. There are three types of commands:
  - A **kernel execution command** executes a kernel on a compute unit
    of a compute device.
  - A **memory command** transfers data to, from, or between memory
    objects, or map and unmap memory objects from the host's address space.
  - A **synchronization command** constrains the order of execution of
    commands.

* Commands in the command queue can be executed in two modes:
  - **In-order execution**: The commands are executed in the order
    that they are placed into the command queue. A command must complete
    before the next one begins.
  - **Out-of-order execution**: Commands are launched in the order,
    but a command can execute without waiting for completion of prior
    commands. Specific ordering constraints on the execution of
    commands can be enforced by explicitly issuing synchronization
    commands.
  
