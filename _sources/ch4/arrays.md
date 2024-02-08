(sec:arrays)=
# Arrays

* Vitis HLS implements arrays (except for array arguments of the
   top-level function) as some type of memory during synthesis. The
   types of memory supported include, RAM (`RAM_1P`, `RAM_1WNR`,
   `RAM_2P`, `RAM_S2P`, `RAM_T2P`), ROM (`ROM_1P`, `ROM_2P`,
   `ROM_NP`), and shift registers (`FIFO`).  More detailed
   descriptions about these different types of memory can be found
   from the manual of [`#pragma HLS
   bind_storage`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-bind_storage). Each
   of these types of memory can be implemented using different PL
   resources, such as block RAM, URAM, and CLBs.
   ```{tip}
   Since arrays are synthesized as memory, they can only be
   instantiated in the C/C++ code with sizes known during
   synthesis.
   ```

* By default, Vitis HLS automatically decides the type of memory to
  synthesize an array into as well as the suitable PL resource to
  implement the selected type of memory. For example, if the Vitis HLS
  infers that an array is ether read or written only once per
  iteration in a loop, then it synthesizes the array as a single-port
  RAM (`RAM_1P`), and if the array size is small, Vitis HLS (actually
  Vivado) implements the single-port RAM will be implemented using
  CLBs in the PL. We may override Vitis HLS by using the bind-storage
  pragma.

* Array arguments of the top-level function in a DSP kernel are mapped
  according to {numref}`top_func_args` and will be discussed in
  a later section.

## Array Access Performance
* Let us recall the simple  loop-unrolling example in {numref}`sec:loop_unroll`:
  ```c++ 
  int acc = 0;
  int x[10];
  Loop: for (int n=0; n<10; n++) { 
  #pragma HLS unroll 
    acc += x[n];
  }
  ```
