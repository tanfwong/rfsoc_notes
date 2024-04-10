# ADC as Signal Source

* Recall from {numref}`sec:hardware`, that the RFSoC4x2 board has 4 RF
  input ports that are connected to 4 ADCs on the XCZU48DR RFSoC
  device on board. In addition, as discussed in
  {numref}`sec:class_platform`, the class Vitis extensible platform
  `rfsoc_adc_vitis_platform` configures ADC-D (ADC0 on tile 224) on
  the RFSoC4x2 board at sampling rate 307.2 Msps.

* The main goal of this section is to discuss how to use the ADC
  configured in `rfsoc_adc_vitis_platform` as a signal source to our
  DSP kernels. To this end, we will first review some basic concepts
  pertaining to the sampling theorem. Then, we will talk about how to
  interface with the ADC hardware block in HLS as well as some simple
  efficient HLS coding techniques to process samples from the ADC.
