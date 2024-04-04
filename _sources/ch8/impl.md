(sec:fft_impl)=
# HLS Implementation

## One-shot Implementation
* Let us first consider a naive HLS implementation of the modified
  decimation-in-time butterfly SFG described in
  {numref}`sec:butterfly_mod` (see also {numref}`butterfly8_mod`)

  - Header (`fft.h`):
    ```c++
    #include <ap_fixed.h>
    #include <math.h>
    #include <complex.h>

    #define nu 10         // FFT size M = 2^nu

    const int M = 1<<nu; // FFT size
    const int M2 = M>>1; // M/2

    // typedef template to increase the number of integer 
    // bits going through the FFT butterfly stages
    template <int S>
    using d_t = std::complex<ap_fixed<S+24, S+2> >;

    void top(std::complex<float> *in, std::complex<float> *out);
    ```
  * Kernel Source (`fft.cpp`):
    ```c++
    #include "fft.h"

    #define BTFY_PARA 1  // Parallelization factor in each butterfly stage

    static d_t<nu> w[M2];
    static int br[M];


    // Set up LUTs for twiddle factors and bit reversal indices
    void init_twiddle_table(d_t<nu> *tw) {
      for (int k=0; k<M2; k++) {
        double c = cos(2*M_PI*k/M);
        double s = -sin(2*M_PI*k/M);
        tw[k] = d_t<0>(c, s);
      }
    }

    void init_bit_reversal_table(int *bit_reverse) {
      bit_reverse[0] = 0;
      bit_reverse[M-1] = M-1;
      for (int k=1; k<M-1; k++) {
        int pattern = k;
        bit_reverse[k] = pattern & 1;
        for (int i=1;i<nu; i++) {
          bit_reverse[k] <<= 1;
          pattern >>= 1;
          bit_reverse[k] |= pattern & 1;
        }
      }
    }

    void butterfly_stage(int i, d_t<nu> *in, d_t<nu> *out) {
    #pragma HLS inline off
    #pragma HLS function_instantiate variable=i
      // Setting up masks 
      int step = 1<<i;
      int lmask = step-1;
      int umask = ~lmask;
      // Going over the M/2 basic butterflies
      Butterfly_Loop: for (int k=0; k<M2; k++) {
    #pragma HLS unroll factor=BTFY_PARA
        int km = k&lmask;
        int idx0 = ((k&umask)<<1) | km;
        int idx1 = idx0 | step;
        km <<= nu-i-1;
        d_t<nu> in0 = in[idx0];
        d_t<nu> in1 = in[idx1];
        if (i>0) in1 *= w[km];
        out[idx0] = in0 + in1;
        out[idx1] = in0 - in1; 
      }
    }

    void fft(d_t<nu> *in, d_t<nu> *out) {
      d_t<nu> X[nu][M];
    #pragma HLS array_partition variable=X dim=1 type=complete
    #pragma HLS array_partition variable=X dim=2 type=cyclic factor=BTFY_PARA

      Reversal_Loop: for (int n=0; n<M; n++) {
    #pragma HLS unroll factor=BTFY_PARA*2
        X[0][n] = in[br[n]];
      }
      Stage_Loop: for (int i=0; i<nu-1; i++) {
    #pragma HLS unroll
        butterfly_stage(i, &X[i][0], &X[i+1][0]);
      }
      butterfly_stage(nu-1, &X[nu-1][0], out);
    }

    void load(std::complex<float> *in, d_t<nu> *buf) { 
      Read_Loop: for (int n=0; n<M; n++) {
        buf[n] = in[n];
      }
    }

    void store(d_t<nu> *buf, std::complex<float> *out) {
      Write_Loop: for (int k=0; k<M; k++) {
        out[k] = buf[k];
      }
    }

    void top(std::complex<float> *in, std::complex<float> *out) {
    #pragma HLS interface mode=m_axi port=in depth=M
    #pragma HLS interface mode=m_axi port=out depth=M

      // Create twiddle (w^k_M) table
      init_twiddle_table(w);
      // Create bit reversal tables
      init_bit_reversal_table(br);
  
      d_t<nu> buf_in[M], buf_out[M];
    #pragma HLS array_partition variable=buf_out type=cyclic factor=BTFY_PARA
  
      load(in, buf_in);
      fft(buf_in, buf_out);
      store(buf_out, out);
    }
    ```

  - The implementation uses complex-valued fixed-point arithmetic
    instantiated by the `d_t<S>` template for the
    `std::complex<ap_fixed<S+W,S+I>>` class. Note that going through
    each basic butterfly element, we need to add one integer bit in the
    fixed-point representation of the output. The `d_t<S>` template
    helps to account for this requirement as we move through the
    stages of the butterfly SFG.
  - Look-up tables are generated to store the twiddle factors and
    bit-reversal indices in the `static` vectors `w` and `br`,
    respectively. By declaring the vector as `static` and writing to
    them only once, Vitis HLS will infer that they should be
    implemented as ROM, and will not synthesize the initialization
    functions `init_twiddle_table()` and `init_bit_reversal_table()`
    but only use them to calculate the ROM values.
  - The butterfly stages with different structures as shown in
    {numref}`butterfly8_mod` are genrally implemented in the function
    `butterfly_stage()` with logics and masks to specify the
    connection patterns in different stages. The loop `Butterfly_Loop`
    goes over the $\frac{M}{2}$ basic butterfly elements in each
    stage. In order to achieve an II=1 for the `Butterfly_Loop` for
    each stage, we need to use the function-instantiate pragma to
    optimize the RTL synthesized to implement the each stage instance
    of `butterfly_stage()`. Since the structures of the stages are
    different, $\nu$ different instances of the function will be
    synthesized. 
  - All the stages in the butterfly SFG are instantiated in the loop
    `Stage_Loop` of the function `fft()`. The loop is fully unrolled,
    and $\nu$ different instances of `butterfly_stage()` will be
    synthesized to implement the butterfly SFG. The two-dimensional
    array `X` is instantiated to hold the intermediate FFT
    coefficients shown in the shaded vertices in
    {numref}`butterfly8_mod`. The array is implemented as blockRAM,
    and is partitioned differently in the two dimensions to improve
    access to the block RAM.
  - We may use more PL resources to parallelize the processing the
    butterfly elements in each stage by partially unrolling
    `Butterfly_Loop`. To gain speedup advantage, the array `X` needs
    to be partitioned with a higher factor.

    ```{warning}
    Since the input and/or output arrays of `load()`,
    `store()`, and different stage instances of  `butterfly_stage()`
    are not sequentially access, we can not implement, using the
    dataflow pragma, task-level
    pipelining across these tasks in the above implementation. That
    means the load task, FFT operations across the stages, and store
    task have to run in sequence.
    ```

* The redrawn modified butterfly SFG discussed in
    {numref}`sec:butterfly_uniform` and {numref}`butterfly8_unif` with
    uniform stages can be implemented by replacing the function
    `butterfly_stage()` in the kernel code above with the following
    version
  ```c++
  void butterfly_stage_uniform(int i, d_t<nu> *in, d_t<nu> *out) {
  #pragma HLS inline off
  //#pragma HLS function_instantiate variable=i
    // Going over the M/2 basic butterflies
    Butterfly_Loop: for (int k=0; k<M2; k++) {
  #pragma HLS unroll factor=BTFY_PARA
      int idx0 = k<<1;
      int idx1 = idx0+1;
      int km = (idx1 >> (nu-i)) << (nu-i-1);
      d_t<nu> in0 = in[idx0];
      d_t<nu> in1 = in[idx1];
      if ((i>0) and (km>0)) in1 *= w[km];
      out[k] = in0 + in1;
      out[k+M2] = in0 - in1; 
    }
  }
  ```
    - One advantage of this implementation of uniform butterfly stages
      is that only two instances of the function
      `butterfly_stage_uniform()` are synthesized by Vitis HLS (one
      for the first $\nu-1$ stages and one for the last stage. This
      significantly reduces the amount of PL resources consumed
      without sacrificing any meaningful latency and throughput
      performance.
    - The uniform stage structure avoids the need of synthesizing
      $\nu$ differently optimized instances of non-uniform stages as
      in the previous implementation. One could force synthesis of $\nu$
      instances of `butterfly_stage_uniform()` by uncommenting the
      line with the function-instantiate pragma. However, doing so
      would not achieve any meaningful latency and throughput
      performance gain.
