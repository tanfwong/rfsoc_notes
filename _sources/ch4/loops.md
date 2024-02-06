(sec:loops)=
# Loops
Loops are standard C/C++ constructs that are ubiquitous in specifying
repetitive operations in a DSP kernel. The two main HLS optimization
techniques for loops are *pipelining* and *unrolling*. Pipelining and
unrolling on loops are simply the instruction-level counterparts of
task-level pipelining and parallelization discussed in
{numref}`sec:pro-con`.

## Loop Pipelining
* By default, Vitis HLS automatically pipelines a loop. We may also
  invoke loop pipelining by using [`#pragma HLS
  pipeline`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-pipeline)
  as shown in the following C++ code snippet from {cite}`ug1399`:
  ```c++
  vadd: for(int i = 0; i < len; i++) {
  #pragma HLS pipeline II=1
    c[i] = a[i] + b[i];
  } 
  ```
   
  ```{tip}
  It is recommended that we label each loop in the C++ code to 
  ease the processing and reporting processes by Vitis HLS.
  ```

  - The pipeline pragma above tells Vitis HLS to target at achieving an II
     of one clock cycle for the pipeline.
  - Suppose it takes one clock cycle to read `a[i]` and `b[i]`, one
     clock cycle to add them, and one clock cycle to write to
     `c[i]`. Without the pipeline pragma, each iteration of loop
     will take 3 clock cycles and the loop's II=3.
  - Since there is no ***loop dependence***, i.e., the next iteration
     of the loop may begin before the current iteration completes, the
     target II=1 can be achieved by pipelining the loop `vadd` as
     shown in the example above.


* Hardware constraints, data dependencies within an iteration, and/or
  loop dependencies may prohibit Vitis HLS to achieve the specified II
  value for a loop.
  ```{tip}
  * If the target II value specified in the pipeline pragma can not be
  achieved, Vitis HLS will produce a design with the specified II
  value and issue a timing vilation warning message during synthesis. 
  * Not specifying the `II=1` option in the pipeline pragma also sets
  the target II to 1; however by not specifying the target II, 
  Vitis HLS will generate a design that achieves the lowest possible
  II and issue an II violation message instead.
  ```
* In some cases, we may be able to re-factor the C++ code forming the
   loop to achieve a lower II. For example, consider the loop `iir1`
   below that implements a simple first-order IIR filter:
  ```c++
  y[0] = x[0];
  iir1: for (int n=1; n<100; n++) {
  #pragma HLS pipeline II=1
    y[n] = y[n-1]/2 + x[n];
  }
  ```
  where `x` and `y` are both `int` arrays containing 100 elements. The
  loop `iir1` carries a type of loop dependence called *cyclic
  dependence* where the current iteration of the loop depends on the
  results obtained in the previous iterations. The cyclic dependence
   could render the specified II=1 not achievable in the `iir1`
   loop. Nevertheless, consider re-factoring the loop as below:
  ```c++
  y[0] = x[0];
  int tmp = y[0];
  iir2: for (int n=1; n<100; n++) {
  #pragma HLS pipeline II=1
    tmp = tmp/2 + x[n];
    y[n] = tmp;
  }
  ```
  Assume that each iteration in `iir2` can be completed in 3 clock cycles:
  - In the first clock cycle, `x[n]` is read.
  - In the second clock cycle, the division of `tmp`, the addition
      with `x[n]`, and the update of `tmp` in the line 
      `tmp = tmp/2 + x[n]` can all be done. (Note that `tmp` is
      instantiated as a register rather than RAM and thus can speed
      the operations in this line.)
  - In the third clock cycle, `y[n]` is written. 
  
  Then, the specified II=1 can be achieved in spite of the cyclic
  dependence. As a matter of fact, Vitis HLS is smart enough to
  perform the re-factoring for `iir1` automatically during
  synthesis. See Lab 3 for a more in-depth treatment of the above
  example.

* We may explicitly turn off loop pipelining by putting `#pragma HLS
  pipiline off` in the loop body.

* We may *rewind* a pipelined loop to effect continuous execution of
  successive calls to the loop by using the option `#pragma HLS
  pipeline rewind`.  Rewinding can only apply if there is one single
  loop inside the top-level function and the code segment before the
  loop is executed only once in the pipeline without any conditionals.

* We may also specify the type of pipeline to be used using the
  `style=` option. The three possible choices are `stp` standing for a
  stall pipeline, `flp` standing for a flushable pipeline, and `frp`
  standing for a free-running pipeline. The stall pipeline is the
  default choice. See {cite}`ug1399` for details about these three
  different pipeline types.

* We may also pipeline the body of a function in the same way that we
  pipeline a loop.

## Loop Unrolling
* Unrolling a loop creates multiple copies of the loop body for
  parallelization. Clearly, loop unrolling can reduce the iterative
  latency of the loop at the expense of increasing PL resource
  utilization. 

* We may use [`#pragma HLS
  unroll`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-unroll)
  in a loop body to tell Vitis HLS to unroll the loop. 
  - By default, Vitis HLS keeps all loops rolled unless the unroll
    pragma is specified.
  - Specifying the pragma with no option will fully unroll the loop
    and remove the loop hierarchy. In order the fully unroll a loop,
    the loop bound must be known (and fixed) at the build time.
  - We may also partially unroll a loop by specifying the `factor`
    option in the unroll pragma. For example, `#pragma HLS unroll
    factor=2` creates 2 copies of the loop body and reduces the trip
    count of the loop to a half of the original value. The loop bound
    can be variable for a loop to be partially unrolled. In such case
    and whenever necessary,
    Vitis HLS will add control logic to perform the necessary loop
    exit check. We can turn off adding the exit check logic by using
    the `skip_exit_check` option.

* For example, by fully unrolling the following loop: 
  ```c++ 
  int acc = 0; 
  Loop: for (int n=0; n<10; n++) { 
  #pragma HLS unroll 
    acc += x[n];
  } 
  ``` 
  Vitis HLS generates a tree of binary adders with a depth of
  $\lceil \log_2 10 \rceil = 5$. However, we may not see much reduction
  in the latency of `Loop` because of limitations in accessing the
  elements of array `x` which is stored in RAM. See more discussions
  about this in {numref}`sec:arrays` and Lab 3. 

## Loop Merging
* Each loop creates at least 1 state in the FSM of a DSP kernel. If
  there are a sequence of loops in a function of the C++ specification
  of the kernel, it takes at least one clock cycle to enter a loop,
  another clock cycle to exit the loop and enter the next loop in the
  sequence, and so on. *Loop merging* is the optimization that
  merges consecutive loops into a single loop to reduce overall
  latency, and to allow potential control logic sharing logic as well
  as parallelization if possible.

* Using [`pragma HLS
  loop_merge`](https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/pragma-HLS-loop_merge)
  tells Vitis HLS to merge all loops within the scope the pragma is
  placed, conforming to the following set of rules:
  - If the loop bounds are variables, they must have the same value,
    i.e., the number of iterations of the loops mus be the same.
  - If the loop bounds are constants, the maximum is used as the bound of the merged loop.
  - Loops with both variable bounds and constant bounds cannot be merged.
  - The code between loops to be merged cannot have side effects,
    i.e., multiple execution of this code should generate the same results.
  - Loops cannot be merged when they contain FIFO reads because
    merging may change the order of the reads.


## Nested Loops
