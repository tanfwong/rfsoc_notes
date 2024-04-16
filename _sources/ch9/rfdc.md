# ADC Interface

* As discussed in {numref}`sec:zynq`, the XCZU48DR RFSoC device
  contains hardened data converter blocks and PLLs to support the ADCs
  (and DACs) on chip:
  ```{figure} ../figs/adc_tile.png
  ---
  name: adc_tile
  alt: ADCs with supporting data converter and PLL blocks
  width: 800px
  align: center
  ---
  Block diagram of an ADC tile with supporting data converter and PLL
  blocks on the XCZU48DR RFSoC device (image taken from {cite}`pg269`)
  ```
* The ADC portion of the data converter block implements a number of
  DSP functions as shown in the figure below, including: 
  ```{figure} ../figs/rfadc.png 
  --- 
  name: rfadc 
  alt: diagram of the ADC portion of the data converter block 
  width: 1000px 
  align: center 
  --- 
  Block diagram of the ADC portion of the data converter
  block on the XCZU48DR RFSoC device (image taken from {cite}`pg269`)
  ```
  - a signal magnitude detector, 
  - a quadrature modulator correction (QMC) block,
  - a Digital Down Converter (DDC) that consists of
    - coarse frequency mixers and a numerically controlled oscillator
      (NCO), and 
    - signal decimators with aliasing filters.

* All these DSP function components can be configured to implements
  standard Nyquist sampling (in the first Nyquist zone) of a
  real-valued baseband signal as discussed in {numref}`sec:oversample`
  and second Nyquist-zone sampling of a real-valued bandpass signal as
  discussed in {numref}`sec:nyquistzone`. Other modes of sampling,
  including sampling of complex-valued baseband and bandpass signals
  from the in-phase (I) and quadrature (Q) signal paths, can also be
  implemented using the hardened DSP functions.

* The configuration of the DSP functions can be set when building the
  Vitis extensible platform using the RFDC IP block {cite}`pg269`. In
  `rfsoc_adc_vitis_platform`, the configuration is chosen to implement 
  Nyquist sampling of a real-valued baseband signal.
  
* The sampling rate of the ADCs is set based on the frequency of the
  stable reference clock input provided on the RFSoC 4x2 board. In
  `rfsoc_adc_vitis_platform`, it is set to $4.9152$ Gsps. The DDC in
  the data converter block allows us to decimate the ADC output in
  order to equivalently lower the sampling rate (see [my DSP
  notes](https://tanfwong.github.io/dsp_notes/ch7/down.html) for a
  more detailed discussion). In
  `rfsoc_adc_vitis_platform`, the decimation factor is set to 16,
  resulting in the sampling rate of $307.2$ Msps reported in
  {numref}`sec:class_platform`. 

* The ADCs on XCZU48DR RFSoC device have a resolution of 14 bits (see
  {numref}`sec:zynq`). Each ADC sample is provided as a 16-bit
  fixed-point/integer value. The data converter block contains FIFOs
  to provide an AXI4 stream (`axis`) interface for our DSP kernel to
  access the stream of samples. Up to 12 samples (see the 192-bit wide
  data path in {numref}`rfadc`) can be packed together as the basic
  unit of the `axis` stream to reduce the clock rate required to
  support the `axis` interface. In `rfsoc_adc_vitis_platform`, eight
  samples are packed into a chunk for `axis` streaming, requiring a
  minimum clock rate of $38.4$ MHz for the `axis` interface. The data
  converter block can be configured to provide a reference clock at
  that frequency to drive the `axis` interface as shown in
  {numref}`hardware`.

* Below is a simple HLS kernel example that reads chunks of samples
  from the `axis` interface of the data converter block and then
  stores them in the global memory:

  Kernel header (`stream_to_mem.h`):
  ```c++
  #include <ap_fixed.h>
  #include <hls_stream.h>
  #include <tuple>

  #define MAX_N 8192   // Number of samples
  #define C 8  // Number of samples per chunk
  #define MAX_NC MAX_N/C

  // Basic ADC sample type
  typedef ap_fixed<16,1> d_t;
  // Chuck type = array of C samples
  typedef std::array<d_t,C> c_t;


  extern "C" void top(hls::stream<c_t> &s_in, c_t *out, unsigned long N);
  ```

  Kernel:
  ```c++
  #include "stream_to_mem.h"
  #include <assert.h>

  void store(hls::stream<c_t> &in, c_t *out, unsigned long N) {
    assert(N%4==0);
    Write_Loop: for (unsigned long n=0; n<N; n++) {
  #pragma HLS loop_tripcount max=MAX_NC
      out[n] = in.read();
    }
  }

  extern "C" {
  void top(hls::stream<c_t> &s_in, c_t *out, unsigned long N) {
  #pragma HLS interface mode=axis port=s_in depth=MAX_NC
  #pragma HLS interface mode=m_axi port=out depth=MAX_NC
  #pragma HLS dataflow
  
    store(s_in, out, N/C);
  }
  }
  ```
  - The 16-bit samples from the data converter are casted into the
    `ap_fixed<16,1>` type. 
  - The same technique of chunking using the `std::array` class in
    {numref}`sec:blk-by-blk-fft` is employed here.
  - A `hls::stream` input argument is employed in the top-level
    function `top()` to interface with the `axis` sample stream
    provided by the data converter block.
  - Chunks of fixed-point samples are stored in the global memory
    as the output of the kernel.
  
  Host code snippet:
  ```c++
  // Compute the size of array in bytes
  size_t size_in_bytes = NC*sizeof(c_t);
  // Instantiate host input and output vectors
  std::vector<c_t, aligned_allocator<c_t> > x(NC);

  // These commands will allocate memory on the Device
  // and link to host pointers
  OCL_CHECK(err, cl::Buffer x_buf(context, 
    CL_MEM_USE_HOST_PTR|CL_MEM_WRITE_ONLY, size_in_bytes, x.data(), &err));

  // set the kernel Arguments
  unsigned long numsamps = N;
  OCL_CHECK(err, err = krnl.setArg(1, x_buf));
  OCL_CHECK(err, err = krnl.setArg(2, numsamps));

  OCL_CHECK(err, err = q.enqueueTask(krnl));
  // Transfer output from gloabl to host memory
  OCL_CHECK(err, err = q.enqueueMigrateMemObjects({x_buf}, 
    CL_MIGRATE_MEM_OBJECT_HOST));
  OCL_CHECK(err, err = q.finish());
  std::cout << "Done getting signal sample from ADC.\n";

  // save output samples to file
  std::cout << "Writing data to signal.txt\n";
  std::ofstream file;
  file.open("signal.txt");
  for (int n=0; n<N; n++)
    file << x[n/C][n%C] << std::endl;
  file.close();
  ```
  - Only the top-level function arguments of the output global memory
    buffer and the number of samples to capture are set in the host
    code.
  - Explicit connection of the `hls::stream` argument of the top-level
    function to the `axis` interface of the data converter block must
    be specified in the kernel configuration file in Vitis (see Lab
    9).
  - If the HLS kernel and the data converter block's `axis` interface
    are under different clock domains (e.g., in
    `rfsoc_adc_vitis_platform`, the HLS kernel is drived by the $200$
    MHz platform clock while the data converter block's
    `axis` interface clock is at $38.4$ MHz as discussed above), 
    Vitis will automatically insert an AXI4 stream clock converter to
    interface between the kernel and `axis` interface as shown:
  ```{figure} ../figs/kernel_bd.png 
  --- 
  name: kernel_bd 
  alt: Block diagram showing connection between HLS kernel and data converter axis interface
  width: 1000px 
  align: center 
  --- 
  Block diagram showing connection between the HLS kernel and the
  data converter's `axis` interface in `rfsoc_adc_vitis_platform`.
  ```
