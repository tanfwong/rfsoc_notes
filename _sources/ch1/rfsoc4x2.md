# Hardware Platform

We use the [RealDigital RFSoC 4x2
board](https://www.realdigital.org/hardware/rfsoc-4x2) as the hardware
platform for DSP implementation:
```{figure} ../figs/rfsoc4x2_board.jpg
---
name: board
alt: RealDigital RFSoC 4x2 Board
width: 1000px
align: center
---
RealDigital RFSoC 4x2 Board (image taken from
{cite}`rfsoc4x2_manual`) 
```

## Board Specifications
* We briefly summarize the relevant specifications of the RFSoC 4x2
  board here. The information and figures are taken directly from
  {cite}`rfsoc4x2_manual`.

* The RFSoC 4x2 board is built around the [Zynq XCZU48DR UltraScale+
RFSoC
device](https://docs.xilinx.com/v/u/en-US/ds889-zynq-usp-rfsoc-overview)
from AMD-Xilinx with radio-frequency (RF) input (4) and output (2)
ports, stable clock references, 8 GB DDR4 memory, and a number of standard
peripherals and ports for a typical embedded system development board:
  ```{figure} ../figs/rfsoc4x2_diagram.jpg
  ---
  name: board_diagram
  alt: RealDigital RFSoC 4x2 Board Diagram
  width: 600px
  align: center
  ---
  Block diagram of the RFSoC 4x2 Board (image taken from
  {cite}`rfsoc4x2_manual`)
  ```

* We will primarily utilize the Zynq RFSoC device, the clock
  references, the RF ports, and the DDR4 memory for DSP development in
  this class.

## Zynq RFSoC

* The Zynq XCZU48DR UltraScale+ RFSoC device implements a
  heterogeneous system consisting of the following component subsystems:
  1. Processing System (PS):
    - a 64-bit quad-core ARM Cortex-A53 
    - a 32-bit dual-core ARM Cortex-R5F
    - an ARM Mali-400 based GPU and NEON advanced SIMD media processing engine
    - an 8-channel DMA controller that supports 64-bit, 2400MHz DDR4
    - connectivity controllers for PCI Express, SATA, DisplayPort, Gbit Ethernet, USB3 and other ports
    - a platform management unit 

  2. Programmable Logic (PL):
    - 

  3. RF System:
    - eight 14-bit RF ADCs supporting up to 5.0 Gsps sampling rate
      (the RFSoC 4x2 board provides SMA input ports to 4 ADCs)
    - eight 14-bit RF DACs supporting up to 9.85 Gsps sampling rate
      (the RFSoC 4x2 board provides SMA output ports to 2 DACs)
    - eight SD-FEC hardened IP blocks
    - hardened IP blocks supporting up to 40x decimation/interpolation
  
  ```{figure} ../figs/rfsoc_g3.png
  ---
  name: xczu48dr
  alt: Zynq XCZU48DR UltraScale+ RFSoC Board Diagram
  width: 1000px
  align: center
  ---
  Block diagram of the Zynq XCZU48DR RFSoC device (image taken from
  [this Xilinx
  document](https://docs.xilinx.com/r/en-US/ug1496-t2-telco/Zynq-UltraScale-RFSoC-Specification))
  ```

