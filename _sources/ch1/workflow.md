(sec:vaad)=
# Vitis Application Acceleration Development

* As discussed in {numref}`sec:vitis`, we have to program the
  different compute components and construct interfaces (both hardware
  and software) between them in the typical embedded system
  development process. In particular, the development of the AXI4
  interfaces between the DSP kernels and the DMA interface with the
  global memory can be rather laborious. While the implementation of
  these interfaces may serve as good design exercises in a digital
  design class, it is not exactly the focus of what we want to do
  here, namely DSP kernel implementation.

* In order to simplify our development process, we will employ the
  following approach:
  1. Follow the *Vitis Application Acceleration Development (VAAD) Flow*. 
  2. Use HLS to develop DSP kernels under the VAAD flow.  We will
     discuss more about the HLS design process in {numref}`sec:hls`.

## Vitis Application Acceleration Development Flow
* The VAAD flow {cite}`ug1393` is essentially a design process that
  allows developers to focus on developing the core functionalities of
  the PL kernels while the development of the required interfaces is
  all automated by Vitis.

* It contains:
  - a *Vitis extensible platform* which serves as an abstraction of
    the PL hardware to the host application development process, and
    the platform is composed of
    - a *domain* with the necessary software to run Linux on the PS host
    - a base PL hardware block with a predefined configuration of AXI4
      interfaces for PL kernel control by the PS host and for data
      transfer between the PS host and the PL kernels via global
      memory, clocks, and interrupted signals
    ```{figure} ../figs/vitis_platform.png
    ---
    name: vitis_platform
    alt: Vitis extensible platform
    width: 600px
    align: center
    ---
    Components of a Vitis extensible platform
    (image taken from {cite}`ug1393`)
    ```
  - a predefined configuration of AXI4 interfaces for kernel control
      and data transfer, clock input, and interrupt signals to which
      each PL kernel must conform
  - the [Xilinx Runtime
    (XRT)](https://xilinx.github.io/XRT/2023.2/html/index.html)
    library which contains Linux drivers and APIs to support PL kernel
    control and data transfer in host application programming.

* In addition, Vitis HLS can be employed to automatically synthesize
  interfaces that conform to the predefined configuration required by
  VAAD for a PL kernel. 

* The tradeoffs for adopting VAAD are potential losses in flexibility
  of the implementation architecture and in PL utilization
  efficiency. Nonetheless, these potential losses are rather
  acceptable for us since we will not be interested in developing
  interfaces to other peripherals, and the adoption of VAAD allows us
  to focus only on the "DSP stuff" in our development.

(sec:vaadf_steps)=
## VAAD Procedures in Vitis
The followings are the steps of performing our DSP development under
VAAD:
1. Build a Vitis extensible platform for the RFSoC 4x2 board:
   - Use Vivado to build a hardware platform component
   - Use PetaLinux to build an image for the Linux domain with the
     required boot files, device tree, and XRT library
   - Create a *Platform Component* in Vitis to build the Vitis
     extensible platform

2. Create a *System Project* in Vitis by selecting to use the Vitis
   platform, the Linux kernel image, rootfs, and sysroot generated in
   step 1. The system project will serve as a system container for
   developing our embedded application.

3. Create a *HLS Component* (multiple HLS components) in Vitis to use
   Vitis HLS to develop DSP kernel(s).

4. Create an *Application Component* in Vitis to develop PS host application.

5. Add the HLS and application components to the system project.

6. Build the system project for software emulation, hardware
   emulation, and/or hardware in Vitis. 

7. Deploy the host executable and FPGA bit-stream, if applicable, for
   testing, debugging, and verification. The usual sequence of testing
   build is to start from software emulation, then to hardware
   emulation, and finally to hardware.

* Note that steps 2-5 can be replaced, and often simplified, by
  generating all the components from an *Acceleration Example* in
  Vitis and modifying the corresponding components in the example.

```{figure} ../figs/vitis_build.png
---
name: vitis_build
alt: Vitis extensible platform
width: 1000px
align: center
---
VAAD Procedures in Vitis
(image modified from {cite}`ug1393`)
```

## Class Vitis Extensible Platform
* To ease the development process, I have built a simple Vitis
  extensible platform described in step 1 of {numref}`sec:vaadf_steps`
  for use in class. The platform is called `rfsoc_adc_vitis_platform`
  and will be provided. Those interested may follow the steps similar
  to those in [this
  tutorial](https://github.com/tanfwong/rfsoc4x2/blob/main/vitis_adc_platform.md)
  to generate the Vitis platform.

* The `rfsoc_adc_vitis_platform` has ADC-D (ADC0 on tile 224) on the
  RFSoC4x2 board enabled at sampling rate $307.2$ Msps. It also
  provides two clocks at $200$ MHz and $400$ MHz to drive the PL
  kernels. The block diagram of the hardware platform component
  generated by Vivado is given below:
  
```{figure} ../figs/rfsoc_adc_block_design.png
---
name: hardware
alt: Hardware component of `rfsoc_adc_vitis_platform`
width: 1000px
align: center
---
Hardware platform component of `rfsoc_adc_vitis_platform`
```
