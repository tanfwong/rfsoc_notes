# Data Flow Graph Execution
As discussed in {numref}`sec:pro-con`, it is preferable to follow the
producer-consumer model in the HLS development of a DSP kernel by
decomposing the kernel into a network of tasks connected together by
streaming buffers to form a data flow graph.  Communications and
synchronization between the tasks are through the streaming buffers,
which are also often referred to as ***channels***.

(sec:task_model)=
## Task Model
* To support this data flow graph architecture, each task should be
  constructed in accordance to the model depicted in the figure below:
  ```{figure} ../figs/task.jpg
  ---
  name: task
  alt: task
  width: 800px
  align: center
  ---
  Task Model.
  ```
  Based on this model, a task consists of:
  - an *execution unit* which performs the DSP function of the task,
  - some local memory (BRAM, URAM, and/or registers) that stores data
    accessible only within the task, and
  - a collection of input and output ports that respectively connect
    to streaming buffers (channels) and other connections input to and
    output from the task.

* To conform to the task model in {numref}`task`, a task can be
  specified by a C/C++ function. Vitis HLS infers the various
  components of a task function in the following way:
  - The execution unit is specified by the operations defined in the
    body of the function.
  - The variables in the body of the function are mapped to local
    memory. Vitis determines the types of PL resources to be used for
    different types of variables. Registers and RAM are typically used
    to implement scalar variables and arrays, respectively.
  - The arguments of the task function are mapped to I/O ports. Input
    ports for scalar arguments are connected to registers and ports
    for arrays and streams are connected to streaming buffers. The
    directions of the ports are inferred from the read/write
    operations on the streaming buffers defined in the body of the
    function.

* Vitis HLS supports both *blocking* and *non-blocking* read and write
    semantics for accessing the connected streaming buffers in a
    task. A blocking read from an empty buffer holds the reading
    process until the buffer becomes non-empty. A non-blocking read
    from an empty buffer returns uninitialized or the last read data.
    Similarly, a blocking write to a full buffer holds the writing
    process until the buffer becomes not full. A non-blocking write to
    a full buffer results in the data being dropped.

* Vitis HLS supports a data-driven model and a control-driven model to
  structure the execution of tasks in a data flow graph
  {cite}`ug1399`. Details of these models will be discussed below. The
  use of the two models can also mixed in a data flow graph.

## Data-driven Execution Model
* Under the data-driven model, each task waits for and then executes
  when there is input data for it to process. The tasks in the data
  flow graph do not need to be controlled by any actions, such as
  function calls and data transfer, of the PS host.

* We need to instantiate data-driven tasks as `hls::task` class
  objects and connect them using streaming buffers to specify the data
  flow graph explicitly in the C++ specification of the DSP
  kernel. Only FIFOs, declared using the templatized C++ classes
  `hls::stream<type, depth>` and `hls::stream_of_blocks<block_type,
  depth>`, may be used as streaming buffers for data-driven tasks.

* The use of FIFOs and explicit connections of the data flow graph in
  the data-driven model allows Vitis HLS to infer opportunities for
  pipelining and parallelization when converting the C++ specification
  to RTL.

* For an illustrative example of how to construct a data flow graph
 operating under the data-driven model, consider the C++ code snippet
 (taken from [this
 example](https://github.com/Xilinx/Vitis-HLS-Introductory-Examples/blob/master/Task_level_Parallelism/Data_driven/simple_data_driven/test.cpp))
 below
  ```c++
  void splitter(hls::stream<int>& in, hls::stream<int>& odds_buf,
                hls::stream<int>& evens_buf) {
  int data = in.read();
  if (data % 2 == 0)
    evens_buf.write(data);
  else
    odds_buf.write(data);
  }
  
  void odds(hls::stream<int>& in, hls::stream<int>& out) {
    out.write(in.read() + 1);
  }

  void evens(hls::stream<int>& in, hls::stream<int>& out) {
    out.write(in.read() + 2);
  }

  void odds_and_evens(hls::stream<int>& in, hls::stream<int>& out1,
                      hls::stream<int>& out2) {
  hls_thread_local hls::stream<int, N / 2> s1; // channel connecting t1 and t2
  hls_thread_local hls::stream<int, N / 2> s2; // channel connecting t1 and t3

  // t1 infinitely runs func1, with input in and outputs s1 and s2
  hls_thread_local hls::task t1(splitter, in, s1, s2);

  // t2 infinitely runs func2, with input s1 and output out1
  hls_thread_local hls::task t2(odds, s1, out1);

  // t3 infinitely runs func3, with input s2 and output out2
  hls_thread_local hls::task t3(evens, s2, out2);
  }
  ```
  that specifies the following data-driven data flow graph:
  ```{math}
  :label: even_odd
  \begin{equation}
  \boxed{\text{in}} \rightarrow t_1 \
  \begin{array}{c} 
  \stackrel{s_1}{\nearrow} {}^{\displaystyle t_2 \rightarrow 
  \boxed{\text{out}_1}} \\
  \stackrel{s_2}{\searrow} {}_{\displaystyle t_3 \rightarrow 
  \boxed{\text{out}_2}} 
  \end{array} 
  \end{equation}
  ```
  - The function `odds_and_evens()` is the top-level function of the
    DSP kernel. It has an input FIFO `in` and two output FIFOs `out1`
    and `out2`. Connections of these I/O FIFOs to outside hardware
    components are specified using the Vitis tool. For example, we
    may connect the input FIFO `in` to the ADC activated in the class
    Vitis platform described in {numref}`sec:class_platform`.
  - The data-driven tasks `t1`, `t2`, and `t3` are instantiated as
    `hls::task` class objects as discussed above. The three tasks are
    specified by the functions `splitter()`, `odd()`, and `even()`,
    respectively. The instantiation of the data-driven tasks is
    similar to that of instantiating threads in standard C++. Indeed,
    the qualifier `hls_thread_local` before each instantiation of an
    `hls::task` object is used to ensure the thread that emulates that
    data-driven task starts only once and keeps the same state when
    called multiple times in the C simulation of the HLS code above.
  - The data flow graph in {eq}`even_odd`is explicitly connected by
    setting the `hls::stream<int>` objects `s1` and `s2` as output
    FIFOs for task `t1` and input FIFOs for tasks `t2` and `t3`. Note
    that the qualifier `hls_thread_local` also needs to be used for
    each instantiation of the local (to the DSP kernel)
    `hls::stream<int>` objects.
  
## Control-driven Execution Model 
* Under the control-driven model, execution of tasks in a DSP kernel
  is controlled by the PS host through interactions with the kernel,
  such function calls and parameter passing. Tasks may also access
  global memory in this model.

* Unlike in the data-driven case, we do not need to explicitly
  instantiate the tasks and connect them to form the data flow
  graph. Instead, we may use regular sequential C++ semantics to
  specify the DSP kernel and indicate using the *dataflow*
  pragma/directive the code region for which we want Vitis HLS to
  infer and construct an efficient (acyclic) data flow graph with
  pipelining and parallelization.

* Decomposing the C++ specification of the *dataflow region* into a
  sequence of task functions conforming to the task model in
  {numref}`sec:task_model` can help Vitis HLS to infer a more
  optimized data flow graph. In particular, it is recommended that the
  dataflow region should be written in the *canonical* form
  {cite}`ug1399` for more predictable inferencing by Vitis HLS.

* A dataflow region is in the canonical form if:
  - The task functions should not be inlined. 
  - Each task function's return type must be `void`.
  - Each task function should only use local and non-static variables. 
  - The sequence of task functions should pass data forward such that
    an acyclic data flow graph can be inferred from the
    sequence. Standard C++ scalar and array arguments can be employed
    to pass data from one task function to the next in the
    sequence. The array arguments will be mapped to streaming buffers
    (FIFOs or PIPOs). If a cyclic data flow graph is required, the
    feedback connections must use `hls::stream` or
    `hls::stream_of_blocks` arguments.
  - Array argument variables linking a producer task to a consumer
     task must be written before read.
  - No conditional, loop, return, goto, exception can be used to
    control the data flow in the sequence of task functions. 

* Again, it is more illustrative to consider the following piece of
  C++ code snippet (taken from [this
  example](https://github.com/Xilinx/Vitis-HLS-Introductory-Examples/blob/master/Task_level_Parallelism/Control_driven/Channels/simple_fifos/diamond.cpp))
  showing how to specify a control-driven data flow graph using the
  dataflow pragma:
  ```c++
  typedef unsigned char data_t;

  void diamond(data_t I[N], data_t O[N]) {
    data_t c1[N], c2[N], c3[N], c4[N];
  #pragma HLS dataflow
    A(I, c1, c2);
    B(c1, c3);
    C(c2, c4);
    D(c3, c4, O);
  }
  
  void A(data_t* in, data_t* out1, data_t* out2) {
  #pragma HLS inline off
  Loop0:
    for (int i = 0; i < N; i++) {
  #pragma HLS pipeline
      data_t t = in[i] * 3;
      out1[i] = t;
      out2[i] = t;
    }
  }

  void B(data_t* in, data_t* out) {
  #pragma HLS inline off
  Loop0:
    for (int i = 0; i < N; i++) {
  #pragma HLS pipeline
      out[i] = in[i] + 25;
    }
  }

  void C(data_t* in, data_t* out) {
  #pragma HLS inline off
  Loop0:
    for (data_t i = 0; i < N; i++) {
  #pragma HLS pipeline
      out[i] = in[i] * 2;
    }
  }

  void D(data_t* in1, data_t* in2, data_t* out) {
  #pragma HLS inline off
  Loop0:
    for (int i = 0; i < N; i++) {
  #pragma HLS pipeline
      out[i] = in1[i] + in2[i] * 2;
    }
  }
  ```
  - The top-level function of the DSP kernel is `diamond()`. The
    dataflow region is specified as the scope of the function by the
    dataflow pragma inside the function. The arrays `I` and `O` in the
    function arguments are mapped to global memory. The PS host
    calls this top-level function to start the DSP kernel, and
    transfers data in and out of the kernel by accessing the global
    memory. 
  - The functions `A`, `B`, `C`, and `D` define four tasks. The way
    that the arrays `c1`, `c2`, `c3`, and `c4` enter as the arguments
    of the task functions let Vitis HLS infer the diamond-shape data
    flow graph in {eq}`diamond` with task-level pipelining and the
    independency between tasks $B$ and $C$ for parallelization.
  - By default, the arrays `c1`, `c2`, `c3`, and `c4` are mapped to
    PIPOs (while scalar arguments are mapped to FIFOs). In this
    example, users can also choose to map the arrays to FIFOs as they
    are accessed sequentially as shown in the bodies of the task
    functions.
  - It can be easily check that the dataflow region is specified in
    the canonical form in this example. Instruction-level pipelining
    is also requested by the pipeline pragma in each task function. 

## Mixed Data- and Control-driven model
* We can also mix the two execution models in a data flow graph of a
  DSP kernel. As a matter of fact, since we usually need to save the
  output of our DSP kernel to the global memory, we must use either a
  pure control-driven data flow graph or a mixed-model one. 

* Below is a piece of C++ code snippet specifying a simple
  [example](https://github.com/Xilinx/Vitis-HLS-Introductory-Examples/blob/master/Task_level_Parallelism/Data_driven/mixed_control_and_data_driven/test.cpp)
  of a mixed-model data flow graph:
  
  ```c++
  void worker(hls::stream<int>& in, hls::stream<int>& out) {
    int i = in.read();
    int o = i * 2 + 1;
    out.write(o);
  }

  void read_in(int* in, int n, hls::stream<int>& out) {
    for (int i = 0; i < n; i++) {
      out.write(in[i]);
    }
  }

  void write_out(hls::stream<int>& in, int* out, int n) {
    for (int i = 0; i < n; i++) {
      out[i] = in.read();
    }
  }

  void dut(int in[N], int out[N], int n) {
    hls_thread_local hls::split::round_robin<int, NP> split1;
    hls_thread_local hls::merge::round_robin<int, NP> merge1;
  #pragma HLS dataflow

    read_in(in, n, split1.in);

    // Task-Channels
    hls_thread_local hls::task t[NP];
    for (int i = 0; i < NP; i++) {
  #pragma HLS unroll
      t[i](worker, split1.out[i], merge1.in[i]);
    }

    write_out(merge1.out, out, n);
  }
  ```
  - The top-level function of the kernel is `dut()` with array
    arguments `in` and `out` are mapped to global memory. The dataflow
    region is specified to be the scope of `dut()` within which there
    is a data-driven region specified by the array `t[NP]` of `NP`
    `worker` tasks.
  - The `hls::split` and `hls::merge` class objects `split1` and
    `merge1` are demultiplexing and multiplexing FIFOs, respectively. 
  - For the case of `NP=2`, the data flow graph inferred by Vitis HLS
    is shown below:
  ```{math}
  :label: split_merge
  \begin{equation}
  \boxed{\text{in}} \rightarrow \text{read_in} \ 
  \begin{array}{c} 
  \nearrow {}^{\displaystyle t[0]} \searrow
  \\
  \searrow {}_{\displaystyle t[1]} \nearrow 
  \end{array}
  \ \text{write_out}
  \rightarrow \boxed{\text{out}}
  \end{equation}
  ```

