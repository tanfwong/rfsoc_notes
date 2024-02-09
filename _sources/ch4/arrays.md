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
### Array Partitioning
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
    of `type=cyclic` and `type=block`. See the figure below for
    illustrative examples of the three types of array partitioning:
    ```{figure} ../figs/array_partition.png
    ---
    name: array_partition
    alt: Examples of array partitioning
    width: 800px
    align: center
    ---
    Examples of the three different types of array partitioning where
    the option `factor=2` is set for the cases of `type=block` and 
    `type=cyclic` (image taken from {cite}`ug1399`)
    ```
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
  #pragma HLS array_partition variable=x type=complete
  
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

### Array Reshaping
* An array can be reshaped by using [`#pragma HLS
  array_reshape`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-array_reshape). 
  - The same three types of array partitioning are also available for
    array reshaping with the elements of the original array placed in
    different sections of bits in an element of the reshaped array as
    illustrated by the examples shown in the following figure:
    ```{figure} ../figs/array_reshape.png
    ---
    name: array_reshape
    alt: Examples of array reshaping
    width: 800px
    align: center
    ---
    Examples of the three different types of array reshaping where
    the option `factor=2` is set for the cases of `type=block` and 
    `type=cyclic` (image taken from {cite}`ug1399`)
    ```
   - For a multi-dimensional array, the `dim=` option can be used to
    specify the dimension for which the array should be
    reshaped. Setting `dim=0` reshapes all dimensions.

* If block RAM is chosen to be the PL resource to implement an array,
  array reshaping can reduce the number of block RAM consumed compared
  to array partitioning. However, array reshaping may not be as
  beneficial when `type=complete` because the whole original array is read/or
  write as a single element and extra logic and registers are needed to access the
  individual sections of the element. For example, doing
    ```c++ 
  int acc = 0;
  int x[10];
  #pragma HLS array_reshape variable=x type=complete
  
  Loop: for (int n=0; n<10; n++) { 
  #pragma HLS unroll 
    acc += x[n];
  }
  ```
  gives the same latency for the unrolled loop `Loop` as array
  partitioning but requires more CBN resource to implement the reshaping.

## Array Initialization and Reset
* The standard C++ convention dictates that the elements of an array
  declared within a local scope (e.g., within a function) are not
  initialized. This convention carries over to Vitis HLS.

* If we want to initialize an array, we may use the standard C++ array
  initialization syntax to do so, e.g.,
  ```c++
  int x[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  ```
  ```{tip}
  - Initializing an array as in the example above requires clock cycles
    to write to the array elements to memory. If the array `x` is
    declared (and initialized) with a function and the function is
    called multiple times, then the extra clock cycles consumed on
    initializing `x` are needed for each call to the function.
  - We may save those extra array initialization clock cycles by using
    the qualifier `static` to declare the array, e.g.,
    ~~~c++
    static int x[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    ~~~
    Vitis HLS interprets a `static` array in the same way as in
    standard C++ that the values of the array elements are preserved
    across function calls. In this way, declaring a static array helps
    to better match the behavior of the C++ code with that of hardware 
    memory that implements the array.
  - An additional advantage of declaring a static array is that Vitis
    HLS can hard-code the initialization of its elements into the RTL 
    design so that no extra write clock cycles are needed. 
  ```

* By default, Vitis HLS does not reset static arrays when a hardware
  reset signal is asserted during execution (it does so for a power-on
  reset though). This is done to avoid adding extra reset logic for
  the implementation of the static arrays and the extra
  initialization clock cycles needed. We can overide this default
  configuration on a per array basis by using [`#pragma HLS
  reset`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-reset). 
