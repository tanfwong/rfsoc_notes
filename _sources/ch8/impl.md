(sec:fft_impl)=
# HLS Implementations

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
