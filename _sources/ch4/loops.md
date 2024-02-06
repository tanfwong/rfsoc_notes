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
  
* We may *rewind* a pipelined loop to effect continuous execution of
  successive calls to the loop by using the option `#pragma HLS
  pipeline rewind`.  Rewinding can only apply if there is one single
  loop inside the top-level function and the code segment before the
  loop is executed only once in the pipeline without any conditionals.

* We may also specify the type of pipeline to be used using the
  `style=` option. The three possible choices are `stp` standing for a
  stall pipeline, `flp` standing for a flushable pipeline, and `frp`
  standing for a free-running pipeline. The stall pipeline is the
  default choice. See {cite}`ug1399` for the details about these three
  different pipeline types.

* We may also pipeline the body of a function in the same way that we
  pipeline a loop.

## Loop Unrolling
