# Software Development Tools

(sec:dsp_conf)=
## DSP Implementation Configuration
There are many possible configurations of utilizing the various
compute resources in the three subsystems in the XCZU48DR RFSoC
device to support DSP implementation. Since our main goal is to
learn how to develop real-time, high-speed DSP implementation, we
will employ the heterogeneous compute resources in the XCZU48DR
RFSoC device in the configuration below:
  1. One ADC, the corresponding PLL, and data converter in the RF
     system will be configured to supply a continuous stream of
     (real-valued) signal samples at the sampling rate of $307.2$ Msps.
  2. DSP kernels will be implemented using the various resources in
     the PL.  The results will be stored in the global memory (DDR4)
     connected to the PL.
  3. The ARM Cortex-A53 APU in the PS will serve as a *host* for
     control of the DSP kernels in the PL, interfacing with the global
     memory, and simple post processing of the DSP results. An
     embedded Linux OS will run on the APU.
  4. Control and data signals will be transfered between all these PS,
      PL, and RF components via AXI4 protocols.

(sec:vitis)=
## Vitis-Vivado Toolchain
* DSP implementation in the configuration above follows a typical
  embedded/SoC system development workflow by programming the
  heterogeneous compute components:
  - **PL**:
    1. Configure the ADC, PLL, and data converter to provide a stream
      of input signal samples at the desired sampling rate and
      interface the ADC sample stream to the DSP kernels in the PL.
      This step is usually developed at the *register-transfer level
      (RTL)* using hardware description languages (HDLs), such as
      Verilog and VHDL.
    2. Implement DSP kernels, each with interfaces to the ADC, to other DSP
       kernels, and/or to local and global memory. This step may be developed at
       the RTL level as in the previous step, or with *high-level
       synthesis (HLS)* using  high-level programming languages, such
       as C or C++.
  - **PS**:
    1. Develop drivers and APIs to control the DSP kernels and ADC
      hardware blocks in the PL as well as to transfer data between
      the host (the ARM Cortex-A53 APU) and the DSP kernels.
    2. Develop an application program on the host to control and
       communicate with the DSP kernels and to present the processed
       results from the DSP kernels.

* The AMD-Xilinx tools that support the embedded system
  development workflow is [Vitis](https://www.xilinx.com/products/design-tools/vitis.html), [Vivado](https://www.xilinx.com/products/design-tools/vivado.html), and [Vitis HLS](https://www.xilinx.com/products/design-tools/vitis/vitis-hls.html):
  ```{glossary}
  Vivado
    - Supports RTL design of DSP kernels, hardware blocks, and
      interfaces in the PL
    - Performs FPGA implementation steps, including elaboration,
      verification and simulation, synthesis, place and route,
      static timing analysis, and bit-stream generation

  Vitis HLS
    - Supports HLS design of DSP kernels and interfaces in the PL
    - Performs HLS design steps, including RTL synthesis, C simulation
      and C/RTL co-simulation
    - RTL design sythesized can be used in Vivado for FPGA implementation

  Vitis
    - Supports host application/driver development and build
    - Invokes Vitis HLS and Vivado to perform PL design and implementation
    - Packages the host OS, host application executable, and FPGA bit-stream
      for running the embedded DSP application on the RFSoC 4x2
      board
    - Provids a unified IDE for the embedded system development steps
      above
  ```
