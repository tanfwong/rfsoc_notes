# Overview

* Our goal is to learn how to implement high-speed digital
  signal processing (DSP) objects and algorithms in real time.

* Typical hardware technologies/plaforms/architectures for DSP
  implementation include:
  1. **General-purpose CPU / micro-controller**
  2. **Digital signal processor (dsp)**
  3. **GPU**
  4. **FPGA**
  5. **ASIC**

* Very roughly speaking, the hardware technologies as listed above
  present a decreasing order in *programmability/reconfigurability*
  and an increasing order in *speed*.

* Whether a specific technology is more suitable for a specific
  application also depends on the type of signals to be processed. For
  example, typical GPUs are optimized to process images stored in
  memory but may not be most efficient in processing a continuous
  stream of samples.
   
* In addition, it is a common practice nowadays to have a
  system-on-chip (SoC) device that contains multiple different
  technologies. For example, an SoC device may contain a
  general-purpose CPU for control and interfacing, a GPU for display,
  and an FPGA with *"hardened"* features (effectively ASIC) optimized
  for a specific class of applications.

* In this course, we will focus on the kind of applications in which
  continuous *high-speed* streams of samples coming from
  analog-to-digital converters (ADCs) are to be
  processed. *"High-speed"* here means the sampling rate of the ADCs
  may go up to the giga-samples per second (Gsps) range. However, we
  will be limiting ourselves to about $300$ Msps for easier
  design. Typical applications of this kind include digital
  communications and radar signal processing.

* We will use an SoC device with a programmable logic (PL) component
  in the form of an FPGA and high-speed hardened ADCs as the hardware
  platform to support implementation development for the
  aforementioned DSP applications.
