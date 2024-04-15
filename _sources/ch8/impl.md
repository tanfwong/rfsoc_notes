(sec:fft_impl)=
# HLS Implementation

(sec:1fft)=
## One-shot Implementation
* Let us first consider a naive HLS implementation of the modified
  decimation-in-time butterfly SFG described in
  {numref}`sec:butterfly_mod` (see also {numref}`butterfly8_mod`) for
  calculation of a single $2^\nu$-point FFT:

  Header (`fft.h`):
  ```c++
  #include <ap_fixed.h>
  #include <math.h>
  #include <complex.h>

  #define nu 10        // FFT size M = 2^nu
  const int M = 1<<nu; // FFT size
  const int M2 = M>>1; // M/2

  #define Wb 22
  #define Ib 2
  // typedef template to increase the number of integer 
  // bits going through the FFT butterfly stages
  template <int S>
  using d_t = std::complex<ap_fixed<S+Wb, S+Ib> >;

  void top(std::complex<float> *in, std::complex<float> *out);
  ```
  Kernel Source (`fft.cpp`):
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
      tw[k] = d_t<nu>(c, s);
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
    d_t<nu> X[2][M];
  #pragma HLS array_partition variable=X dim=1 type=complete
  #pragma HLS array_partition variable=X dim=2 type=cyclic factor=BTFY_PARA

    Reversal_Loop: for (int n=0; n<M; n++) {
  #pragma HLS unroll factor=BTFY_PARA*2
      X[0][n] = in[br[n]];
    }
    Stage_Loop: for (int i=0; i<nu-1; i++) {
  #pragma HLS unroll
      butterfly_stage(i, &X[i%2][0], &X[(i+1)%2][0]);
    }
    butterfly_stage(nu-1, &X[(nu-1)%2][0], out);
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
    `std::complex<ap_fixed<S+Wb,S+Ib>>` class. Note that going through
    each basic butterfly element, we need to add one integer bit in
    the fixed-point representation of the output. The `d_t<S>`
    template helps to account for this requirement as we move through
    the stages of the butterfly SFG.
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
    synthesized to implement the butterfly SFG. A ping-pong buffer
    (array `X`) is instantiated to hold the intermediate FFT
    coefficients shown in the shaded vertices in
    {numref}`butterfly8_mod`. The array is implemented as block RAM,
    and is partitioned differently in the two dimensions to improve
    access to the block RAM.
  - We may use more PL resources to parallelize the processing the
    butterfly elements in each stage by partially unrolling
    `Butterfly_Loop`. To gain speedup advantage, the array `X` needs
    to be partitioned with a higher factor.

    ```{caution}
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
    version:
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

  void fft_uniform(d_t<nu> *in, d_t<nu> *out) {
    d_t<nu> X[2][M];
  #pragma HLS array_partition variable=X dim=1 type=complete
  #pragma HLS array_partition variable=X dim=2 type=cyclic factor=BTFY_PARA

    Reversal_Loop: for (int n=0; n<M; n++) {
  #pragma HLS unroll factor=BTFY_PARA*2
      X[0][n] = in[br[n]];
    }
    Stage_Loop: for (int i=0; i<nu-1; i++) {
  #pragma HLS unroll
      butterfly_stage_uniform(i, &X[i%2][0], &X[(i+1)%2][0]);
    }
    butterfly_stage_uniform(nu-1, &X[(nu-1)%2][0], out);
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

* Since access to the input and output arrays of
  'butterfly_stage_uniform()' is not sequential, we still can not
  apply task-level pipelining to the stage instances (if $\nu$
  instances are synthesized) of the function. However, a more careful
  inspection of the modified butterfly SFG in
  {numref}`butterfly8_unif` reveals that if we break up the array
  (`X`) that stores intermediate FFT coefficients in the vertices of
  the SFG into two halves, then access to the half-arrays would be
  sequential. As a result, we may apply task-level pipelining to the
  butterfly stages. The functions `fft_pipelined()` and
  `butterfly_stage_pipelined()` show an example implementation:
  ```c++
  void butterfly_stage_pipelined(int i, d_t<nu> *in0, d_t<nu> *in1, 
    d_t<nu> *out0, d_t<nu> *out1) {
  #pragma HLS inline off
  #pragma HLS function_instantiate variable=i
    // Going over the M/2 basic butterflies
    Butterfly_Loop: for (int k=0; k<M2; k++) {
  #pragma HLS unroll factor=BTFY_PARA
      int idx0 = k<<1;
      int idx1 = idx0+1;
      int km = (idx1 >> (nu-i)) << (nu-i-1);
      d_t<nu> tin0, tin1;
      if (idx0<M2) {
        tin0 = in0[idx0];
        tin1 = in0[idx1];
      } else {
        tin0 = in1[idx0-M2];
        tin1 = in1[idx1-M2];
      }
      if ((i>0) and (km>0)) tin1 *= w[km];
      out0[k] = tin0 + tin1;
      out1[k] = tin0 - tin1;
    }
  }

  void fft_pipelined(d_t<nu> *in, d_t<nu> *out) {
    d_t<nu> X0[nu][M2], X1[nu][M2];
  #pragma HLS array_partition variable=X0 dim=1 type=complete
  #pragma HLS array_partition variable=X0 dim=2 type=cyclic factor=2*BTFY_PARA
  #pragma HLS array_partition variable=X1 dim=1 type=complete
  #pragma HLS array_partition variable=X1 dim=2 type=cyclic factor=2*BTFY_PARA
  #pragma HLS dataflow
    Reversal_Loop: for (int n=0; n<M2; n++) {
  #pragma HLS unroll factor=BTFY_PARA
      X0[0][n] = in[br[n]];
      X1[0][n] = in[br[n+M2]];
    }
    Stage_Loop: for (int i=0; i<nu-1; i++) {
  #pragma HLS unroll
      butterfly_stage(i, &X0[i][0], &X1[i][0], &X0[i+1][0], &X1[i+1][0]);
    }
    butterfly_stage(nu-1, &X0[nu-1][0], &X1[nu-1][0], out, out+M2);
  }
  ```
  - The original array `X` is broken down into two arrays `X0` and
    `X1`, corresponding respectively to the top and bottom halves of
    the vertices in the butterfly stages. `X0` and `X1` are
    partitioned further to support more concurrent access to the block
    RAM in order to achieve an II=1 for the `Butterfly_Loop` in each
    stage.
    ```{caution}
    Because of task-pipelining, we can not just use a ping-pong buffer
    to store the intermediate coefficients calculated by the butterfly
    stages. We need a streaming buffer between each pair of successive
    stages, and hence `X0` and `X1` need to be full-blown two-dimensional
    arrays.
    ```
  - Task-level pipelining on the bit-reversal ordering and the
    butterfly stages is invoked by using the dataflow pragma
    inside the function `fft_pipelined()`and setting the streaming
    buffers to be FIFOs.
  - Although the stage tasks can be pipelined, we may not get the full
    advantage in throughput and latency performance from the
    task-level pipelining because two input coefficients are needed to
    generate each single output coefficient in a stage (see
    {numref}`butterfly8_unif`). Thus, a downstream stage may stall to
    wait for output coefficients generated by its upstream
    counterpart. With task-pipelining, `fft_pipelined()` achieves a
    latency that is about 55\% of the latency of `fft_uniform()`
    without task-pipelining for the case of $\nu=10$.
    ```{caution}
    The stalling behavior is not considered in the synthesis report.
    Hence, the latency performance estimates provided in the synthesis
    report may not be accurate at all. We need to run co-simulation to
    account for the stalling behavior in order to get better latency
    performance estimates. 
    ```
 (sec:blk-by-blk-fft)=  
## Block-by-Block Pipeline Implementation

* In practice, we often take FFT on, sometimes overlapping, blocks from
  a continuous stream of samples. Calling the one-shot FFT implementations
  presented in {numref}`sec:1fft` multiple times is inefficient. 

* A more efficient approach to take blocks of FFT on a continuous
  stream of samples is to pipeline the FFT butterfly stages, for
  example in {numref}`butterfly8_unif`, block by block. 
  
* An example block-by-block pipeline implementation of the
  modified butterfly SFG with uniform stages discussed in
  {numref}`sec:butterfly_uniform` is shown below:
  
  Header:
  ```c++
  #include <ap_fixed.h>
  #include <math.h>
  #include <complex.h>
  #include <tuple>
  #include <hls_stream.h>

  #define nu 10            // FFT size M = 2^nu (nu>=2)
  #define eta 1            // Chunk szie C = 2^eta (eta>=1)
  #define COSIM_NUMBLKS 16 // Number of blocks in COSIM
  #define MAX_NUMBLKS 1000 // Max number of blocks for loop tripcount

  const int M = 1<<nu;  // FFT size
  const int M2 = M>>1;  // M/2
  const unsigned long COSIM_MTOTAL=M*COSIM_NUMBLKS;

  const int C = 1<<eta;  // Number of samples per chunk
  const int C2 = C>>1;   // C/2
  const int MC = M>>eta; // Number of chunks per FFT block
  const int MC2 = MC>>1; // MC/2

  #define MAX_MTOTAL MAX_NUMBLKS*M

  #define Wb 22
  #define Ib 2
  // typedef template to increase the number of integer 
  // bits going through the FFT butterfly stages
  template <int S>
  using d_t = std::complex<ap_fixed<S+Wb, S+Ib> >;
  // typedef template for an array of FFT coeffs
  template <int S, int V>
  using a_t = std::array<d_t<S>, V>;
  // typedef template for a stream of arrays of FFT coeffs
  template <int S, int V>
  using stream_t = hls::stream<a_t<S, V> >;
  // typedef template for an complex float arrays of FFT coeffs
  template <int V>
  using cf_t = std::array<std::complex<float>, V>;

  void top(cf_t<C> *in, cf_t<C> *out, int numblks);
  ```

  Kernel source:
  ```c++
  void pipeline_butterfly_stage(int i, 
    stream_t<nu,C> &in, stream_t<nu,C> &out, int numblks) {
  #pragma HLS inline off
  #pragma HLS function_instantiate variable=i

    // Create a ping-pong buffer
    a_t<nu,M> block[2];
  #pragma HLS array_partition variable=block dim=1 type=complete
  #pragma HLS array_partition variable=block dim=2 type=cyclic factor=C2
    int ping_pong;
    Butterfly_Loop: for (unsigned long k=0; k<numblks*MC; k++) {
  #pragma HLS loop_tripcount max=MAX_MTOTAL/C
      a_t<nu,C> in_chunk = in.read();
      unsigned long kC = k*C;
      int offset = kC%M;
      unsigned long blk = kC>>nu;
      ping_pong = blk%2;

      // Going over C/2 basic butterflies
      Chunk_Loop: for (int j=0; j<C2; j++) {
  #pragma HLS unroll
        int idx0 = j<<1;
        int idx1 = idx0+1;
        int fftk = offset+idx0;
        int km = ((fftk+1) >> (nu-i)) << (nu-i-1);
        fftk >>= 1; 
        if ((i>0) and (km>0)) in_chunk[idx1] *= w[km];
        block[ping_pong][fftk] = in_chunk[idx0] + in_chunk[idx1];
        block[ping_pong][fftk+M2] = in_chunk[idx0] - in_chunk[idx1];
      }
      // Output a chunk from the previous block
      if (blk>0) {
        a_t<nu,C> out_chunk;
        int other_ping_pong = (ping_pong+1)%2; 
        Out_Chunk: for (int j=0; j<C; j++) {
  #pragma HLS unroll
          out_chunk[j] = block[other_ping_pong][offset+j];
        }
        out.write(out_chunk);
      } 
    }
    Output_Last_Block: for (int k=0; k<MC; k++) {
      a_t<nu,C> out_chunk;
      int offset = k*C;
      Out_Chunk_Last: for (int j=0; j<C; j++) {
  #pragma HLS unroll
        out_chunk[j] = block[ping_pong][offset+j];
      }
      out.write(out_chunk);
    }
  }

  void pipeline_bit_reversal_stage(stream_t<nu,C> &in, stream_t<nu,C> &out,
    int numblks) {
  #pragma HLS inline off
    // Create a ping-pong buffer
    a_t<nu,M> block[2];
  #pragma HLS array_partition variable=block dim=1 type=complete
    int ping_pong;
    Bit_Reversal_Loop: for (unsigned long k=0; k<numblks*MC; k++) {
  #pragma HLS loop_tripcount max=MAX_MTOTAL/C
      a_t<nu,C> in_chunk = in.read();
      unsigned long kC = k*C;
      int offset = kC%M;
      unsigned long blk = kC>>nu;
      ping_pong = blk%2;
      Read_A_Chunk: for (int j=0; j<C; j++) {
  #pragma HLS unroll
        block[ping_pong][offset+j] = in_chunk[j];
      }
      // Output a chunk from the previous block
      if (blk>0) {
        a_t<nu,C> out_chunk;
        int other_ping_pong = (ping_pong+1)%2; 
        Reverse_A_Chunk: for (int j=0; j<C; j++) {
  #pragma HLS unroll
          out_chunk[j] = block[other_ping_pong][br[offset+j]];
        }
        out.write(out_chunk);
      } 
    }
    Output_Last_Block: for (int k=0; k<MC; k++) {
      a_t<nu,C> out_chunk;
      int offset = k*C;
      Reverse_A_Chunk_Last: for (int j=0; j<C; j++) {
  #pragma HLS unroll
        out_chunk[j] = block[ping_pong][br[offset+j]];
      }
      out.write(out_chunk);
    }
  }

  void load(cf_t<C> *in, stream_t<nu,C> &buf, int numblks) {
    Read_Loop: for (unsigned long n=0; n<numblks*MC; n++) {
  #pragma HLS loop_tripcount max=MAX_MTOTAL/C
      a_t<nu,C> chunk;
      Chunk_Loop: for (int j=0; j<C; j++) {
  #pragma HLS unroll
        chunk[j] = in[n][j];
      }
      buf.write(chunk);
    }
  }

  void store(stream_t<nu,C> &buf, cf_t<C> *out, int numblks) {
    Write_Loop: for (unsigned long k=0; k<numblks*MC; k++) {
  #pragma HLS loop_tripcount max=MAX_MTOTAL/C
      a_t<nu,C> chunk = buf.read();
      Chunk_Loop: for (int j=0; j<C; j++) {
  #pragma HLS unroll
        out[k][j] = chunk[j];
      }
    }
  }

  void top(cf_t<C> *in, cf_t<C> *out, int numblks) {
  #pragma HLS interface mode=m_axi port=in depth=COSIM_MTOTAL/C
  #pragma HLS interface mode=m_axi port=out depth=COSIM_MTOTAL/C
  #pragma HLS dataflow

    // Create twiddle (w^k_M) table
    init_twiddle_table(w);
    // Create bit reversal tables
    init_bit_reversal_table(br);
  
    stream_t<nu,C> buf_in, buf_stage[nu+1];

    load(in, buf_in, numblks);
    pipeline_bit_reversal_stage(buf_in, buf_stage[0], numblks);
    Stage_Loop: for (int i=0; i<nu; i++) {
  #pragma HLS unroll
      pipeline_butterfly_stage(i, buf_stage[i], buf_stage[i+1], numblks);
    }
    store(buf_stage[nu], out, numblks);
  }
  ```
  - Pipelining of the butterfly stages, bit-reversal stage, load task,
    and store task is implemented using the dataflow pragma in the
    top-level function `top()`. 
  - Complex-valued data samples (coefficients) are packaged into
    chucks of size `C` for streaming to and from the host as well as
    between tasks (stages) for parallelization. 
  - Block-by-block pipelining is explicitly implemented in the
    bit-reversal stage and the butterfly stage task functions
    `pipeline_bit_reversal_stage()` and `pipeline_butterfly_stage()`
    using ping-pong buffers.
  - Latency of `pipeline_butterfly_stage()` mat be further reduced by
    outputting the first half of the coefficients in the butterfly
    stage during the current block as done in
    `butterfly_stage_pipelined()` (see code in {numref}`sec:1fft`).
  - The implementation of `pipeline_bit_reversal_stage()` turns out to
    require a large amount of block RAM (or other memory) resource
    when the value of `C` is set beyond 4. A potential way to overcome
    this problem is to give up the uniform connection structure of the
    butterfly stages and reshuffle the orders of the vertices in each
    stage of the butterfly SFG in order to eliminate the requirement
    of bit-reversal ordering (see {cite}`oppenheim2010`). However,
    this may require different array partition strategies for
    different stages in order to avoid transferring the block RAM
    usage requirement to the butterfly stages instead.




