(sec:loops)=
# Loops
Loops are standard C/C++ constructs that are ubiquitous in specifying
repetitive operations in a DSP kernel. The two main HLS optimization
techniques for loops are *pipelining* and *unrolling*. Pipelining and
unrolling on loops are simply the instruction-level counterparts of
task-level pipelining and parallelization discussed in
{numref}`sec:pro-con`.

## Loop Pipelining
* By default, Vitis HLS automatically pipelines a loop whose 
  *iteration/trip count* is larger than $64$. This trip count
  threshold for automatic pipelining can be set as a configuration
  parameter in Vitis HLS. 

* We may also invoke loop pipelining by using [`#pragma HLS
  pipeline`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-pipeline)
  as shown in the following C++ code snippet from {cite}`ug1399`:
  ```c++
  vadd: for(int i = 0; i < len; i++) {
  #pragma HLS pipeline II=1
    c[i] = a[i] + b[i];
  } 
  ```
  - The pipeline pragma tells Vitis HLS to target at achieving an II
     of one clock cycle for the pipeline (not specifying the `II=1`
     option also set the target II to $1$ as well).
  - Suppose it takes one clock cycle to read `a[i]` and `b[i]`, one
     clock cycle to add them, and one clock cycle to write to
     `c[i]`. Without the pipeline pragma, each iteration of loop
     will take 3 clock cycles and the loop's II=3.
  - Since there is no ***loop dependency***, i.e., the next iteration
     of the loop may begin before the current iteration completes, the
     target II=1 can be achieved by pipelining the loop `vadd` as
     shown in the example above.
 
  ```{tip}
  It is recommended that we label each loop in the C++ code to 
  ease the processing and reporting processes by Vitis HLS.
  ```

* Hardware constraints, data dependencies within an iteration, and/or
  loop dependencies may prohibit Vitis HLS to achieve the specified II
  value for a loop. In such case, Vitis HLS will generate a design
  that achieves the lowest possible II instead. In some cases, we may
  be able to re-factor the C++ code forming the loop to achieve a
  lower II.  In some extreme cases, loop dependencies may render
  pipelining a loop impossible. For example, see the following example
  from {cite}`ug1399`: 
  ```c++ 
  Minim_Loop: while (a != b) {
    if (a > b)
      a -= b; 
    else 
      b -= a; 
  } 
  ``` 
  where each iteration in `Minim_Loop` can
  not start before the previous iteration completes. As a result, this
  loop can not be pipelined.

* We may *rewind* a pipelined loop to effect continuous execution of
  successive calls to the loop by using the option `#pragma HLS
  pipeline rewind`.  Rewinding can only apply if there is one single
  loop inside the top-level function and the code segment before the
  loop is executed only once in the pipeline without any conditionals.
