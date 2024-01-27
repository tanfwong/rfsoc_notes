# Data Flow Graph Execution
As discussed in {numref}`sec:pro-con`, it is preferable to follow the
producer-consumer model in the HLS development of a DSP kernel by
decomposing the kernel into a network of tasks connected together by
streaming buffers to form a data flow graph.  Communications and
synchronization between the tasks are through the streaming buffers,
which are also often referred to as ***channels***.


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
  structure the execution of tasks in a data flow graph. Details of
  these models will be discussed below. The use of the two models can
  also mixed in a data flow graph.

## Data-driven Execution Model
* Under the data-driven model, each task waits for and then executes
  when there is input data for it to process. The tasks in the data
  flow graph do not need to be controlled by any actions, such as
  function calls and data transfer, of the PS host.

* The developer needs to instantiate data-driven tasks as `hls::task`
  class objects and connect them using streaming buffers to specify
  the data flow graph explicitly in the C++ specification of the DSP
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
    respectively. The syntax of instantiation of the data-driven tasks
    is similar to that of instantiating threads in standard
    C++. Indeed, the qualifier `hls_thread_local` before each
    instantiation of an `hls::task` object is used to ensure the
    thread that emulates that data-driven task starts only once and
    keeps the same state when called multiple times in the C
    simulation of the HLS code above.
  - The data flow graph in {eq}`even_odd`is explicitly connected by
    setting the `hls::stream<int>` objects `s1` and `s2` as output
    FIFOs for task `t1` and input FIFOs for tasks `t2` and `t3`. Note
    that the qualifier `hls_thread_local` also needs to be used for
    each instantiation of the local (to the DSP kernel)
    `hls::stream<int>` objects.
  
    

%## Vitis HLS Dataflow Directive
%* The following piece of C++ code snippet shows a simple way to
%  employ the producer-consumer model and the dataflow directive in
%  Vitis HLS to invoke task-level pipelining and parallelization of the
%  diamond-shape data flow graph in {eq}`diamond`:
%  ```c++
%  void diamond(int In[N], int Out[N]) {
%    int c1[N], c2[N], c3[N], c4[N];
%  #pragma HLS dataflow
%    A(In, c1, c2);
%    B(c1, c3);
%    C(c2, c4);
%    D(c3, c4, Out);
%  }
%  ```
%  where each task is written as a function to help the HLS tool to
%  infer the opportunity for pipelining and the independency between
%  tasks $B$ and $C$ for parallelization.
