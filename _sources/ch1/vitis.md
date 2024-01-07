# Software Development Tools

## DSP Implementation Configuration
* There are many possible configurations of utilizing the various
  computing resources in the three subsystems in the XCZU48DR RFSoC
  device to support DSP implementation. Since our main goal is to
  learn how to develop real-time, high-speed DSP implementation, we
  will employ the heterogeneous computing resources in the XCZU48DR
  RFSoC device in the configuration below:
  1. One ADC, the corresponding PLL, and data converter in the RF
     system will be configured to supply a continuous stream of
     (real-valued) signal samples at the sampling rate of $307.2$ Msps.
  2. DSP kernels will be implemented using the various
     resources in the PL. The results will be stored in the global
     memory (DDR4) connected to the PL. 
  3. The ARM Cortex-A53 APU in the PS will serve as a *host* for
     control of the DSP kernels in the PL, interfacing with the global PL memory, and simple post
     processing of the DSP results. An embedded Linux OS will run on
     the APU. 
  4. All these PS, PL, and RF resources will be interconnected via
      AXI4 protocols. 
