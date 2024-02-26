(sec:lcs)=
# Load-Compute-Store Pattern

* The overarching guideline about accessing the global memory that we
  may conclude from the discussions in {numref}`sec:gmem` is that the
  DSP kernel code should be written in a way that enables burst access
  and port widening, preferably automatically inferred and implemented
  by Vitis HLS. In addition, pipeline bursting is often more
  preferable than sequential bursting. 

* In order to help Vitis HLS infer opportunities for pipeling bursting and
  port widening, we may employ the producer-consumer model discussed
  in {numref}`sec:pro-con` to construct a data flow graph connecting
  the following sequence of tasks:
  - a task specializing in loading data from the global memory to
    local buffers,
  - a number of tasks that process the data, and
  - a task specializing in storing processed data back to the global
    memory.

  This coding pattern is referred to as the ***load-compute-store
    (LCS)*** pattern.  
 
