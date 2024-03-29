(sec:lcs)=
# Load-Compute-Store Pattern

* The overarching guideline about accessing the global memory that we
  may conclude from the discussions in {numref}`sec:gmem` is that the
  DSP kernel code should be written in a way that enables burst access
  and port widening, preferably automatically inferred and implemented
  by Vitis HLS.

* In order to help Vitis HLS infer opportunities for pipeling or
  sequential bursting and port widening, we may employ the
  producer-consumer model discussed in {numref}`sec:pro-con` to
  construct a data flow graph connecting the following sequence of
  tasks:
  - a task specializing in loading data from the global memory to
    local buffers,
  - a number of tasks that process the data, and
  - a task specializing in storing processed data back to the global
    memory.

  This coding pattern is referred to as the ***load-compute-store
    (LCS)*** pattern.

* With dedicated *load* and *store* tasks to read from and write to
    the global memory, the code of the load and store tasks can easily
    be written to conform to the conditions for automatic inference of
    bursting stated in {numref}`sec:auto_burst`.

* The compute task(s) on the other hand is (are) free up to access
  only local memory, which not only has shorter latency but also
  provides more flexibility to be structured by us. This is
  particularly beneficial when complex processing steps are to be
  implemented in the DSP kernel.

* For example, consider rewriting the code in the caching example in
  {numref}`sec:cache` in the LCS pattern as below:
  ```c++
  #define N 1000
  
  void load(int in[N], int buf[N]) {
    Read_Loop: for (int n=0; n<N; n++) {
      buf[n] = in[n];
    }
  }

  void compute(int in[N], int out[N]) {
    int past=0;
    Compute_Loop: for (int n=0; n<N; n++) {
      int now = in[n];
      out[n] = past + now;
      past = now;
    }
  }

  void store(int buf[N], int out[N]) {
    Write_Loop: for (int n=0; n<N; n++) {
      out[n] = buf[n];
    }
  }

  void top(int in[N], int out[N]) {
  
    int buf_in[N], buf_out[N];
  
  #pragma HLS dataflow
    load(in, buf_in);
    compute(buf_in, buf_out);
    store(buf_out, out);
  }
  ```
    - Both the `load` and `store` tasks satisfy all conditions stated
      in {numref}`sec:auto_burst`. Thus, Vitis HLS is able to infer
      bursting (Vitis HLS decides to implement sequential bursting in
      this example) and port widening (to 256 bits) for both reads and
      writes of the global memory by the kernel.
    - The `Compute_Loop` is rewritten to achieve an II=1.
    - Task-level optimization (with FIFOs as streaming buffer) is
      applied to the LCS dataflow graph. As a result, the latency of
      the top-level function `top()` less than a half of that
      of the example in  {numref}`sec:cache` is achieved.
      ```{tip}
      The port widening for the writes in `Write_Loop`
      causes a small negative slack in the synthesis step. If that is
      not desirable, we may use the option
      `max_widen_bitwidth=32` in an interface pragma to turn off
      port widening for a more conservative scheduling design.
      ```
 
