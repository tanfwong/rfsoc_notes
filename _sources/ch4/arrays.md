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
  Vitis HLS synthesizes the array `x` into a two-port RAM. Since only two
  reads from the RAM can be made in each clock cycle, it takes 5 clock
  cycles to read the 10 elements in `x` even though `Loop` is unrolled
  into a binary adder tree that can perform the accumulation of the
  10 elements in just a single clock cycle. Indeed, reading from the
  array `x` is in fact the bottleneck of performance in this case.

* Vitis HLS provides two optimizations to solve array access
  bottleneck problems similar to the one in the example above:
  ```{glossary}
  Array Partitioning
    An array is partitioned into smaller arrays or individual
    elements, effectively increasing the number of read and write ports
    to the original array.

  Array Reshaping
    Multiple elements in an array are concatenated to a single element
    with an increased bit-width, allowing more elements in the
    original array to be accessed in a single clock cycle. 
  ```

* An array can be partitioned by using [`#pragma HLS
  array_parition`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-array_partition). 
  - There are three types of ways to partition an array that can be
    specified by the `type=` option in the array-partition pragma:
    ```{glossary}
    `cyclic`
      One element is put into each new smaller array before coming back to the
      first array to repeat the cycle until the array is fully partitioned.

    `block`
      Smaller arrays are formed from consecutive blocks of the original array.

    `complete`
      The original array is decomposed into individual elements, each of
      which is synthesized into a register. 
    ```
  - The number of smaller arrays into which the original array is
    partitioned can be specified by the option `factor=` for the cases
    of `type=cyclic` and `type=block`.
  - For a multi-dimensional array, the `dim=` option can be used to
    specify the dimension for which the array should be
    partitioned. Setting `dim=0` partitions all dimensions.

* Array partitioning clearly improves array access (and hence latency
  and throughput) in the expense of using more PL resources. For
  example, consider completely partitoning the array `x` in the
  previous example:
  ```c++ 
  int acc = 0;
  int x[10];
  #pragma HLS ARRAY_PARTITION variable=x type=complete
  
  Loop: for (int n=0; n<10; n++) { 
  #pragma HLS unroll 
    acc += x[n];
  }
  ```
  Ten registers are synthesized for the 10 elements of `x`. Reading
  from the 10 registers and accumulating through the adder tree
  generated form unrolling `Loop` can all be done in parallel with a
  single clock cycle! The 10 registers with array partitioning and
  two-port RAM without are both implemented using CLB
  resource. Implementation of the former requires more resource.

  
