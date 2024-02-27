# Host Programming

* As discussed in {numref}`sec:dsp_conf`, the PS, more specifically
  the ARM Cortex-A53 APU, serves as a host that controls the DSP
  kernels implemented in the PL. The ADC hardware component serves as
  a data source for the DSP kernels. In addition, the PS may also
  serves as a data source as well as a data (result) sink for the DSP
  kernels. Data transfer between the PS host and the DSP kernels
  occurs via the global memory as discussed in {numref}`sec:gmem`.

* Under the VAAD flow, as discussed in {numref}`sec:vaad`, the XRT
  library provides APIs to let the PS host program control the DSP
  kernels as well as transfer data to and from the kernels via the
  global memory.

* The XRT library provides two different classes of APIs:
  - [native XRT APIs](https://xilinx.github.io/XRT/master/html/xrt_native_apis.html)
  - [OpenCL
    APIs](https://xilinx.github.io/XRT/master/html/opencl_extension.html)
    (Vitis and XRT support OpenCL 1.2 {cite}`opencl1.2`)

* We will discuss mainly the OpenCL APIs here.  We will start with a
   very brief overview of OpenCL and then discuss how to employ the
   OpenCL APIs to set up, control, and execute DSP kernels in the PL
   as well as to perform data exchange with between the host and the
   DSP kernels.
