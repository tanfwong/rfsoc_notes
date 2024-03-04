# OpenCL Optimization

* In many practical scenarios, the DSP kernel may process a long
  sequence of input samples (data) and produce a long sequence of
  output samples (data). As the PL has limited memory resources, the
  long sequence of input samples will need to be broken down into
  shorter blocks. Each shorter input block is passed from the host to
  the DSP kernel for processing and then the output block is passed
  back from the kernel to the host. This process repeats block by
  block until all the blocks generated from the original long input
  sequence are processed.

* This block processing approach necessitates the step sequence of
  transferring input data from the host memory to the global memory,
  executing the kernel to process the data, and transferring the
  output data from the global memory back to the host memory (steps
  7-9) be repeatedly applied from block to block.

* Since the latency of data transfer and time overhead of launching
  the DSP kernel are significantly, sequentially repeating the step
  sequence from block to block may significantly reduce the processing
  throughput. 

* In order to overcome this inefficiency, we need to optimize the OpenCL
  host code to support pipelining the steps of data transfer and
  kernel execution as well as parallelization with multiple compute
  units in the PL.

(sec:data_transfer)=
## Data Transfer
* Under the OpenCL memory model, data exchange between the host and
  the kernels amounts to synchronization of contents between the host
  memory and the global memory. As the latency of this memory
  synchronization process is significant, it is important to optimize
  the performance of  this process.

* The first step of data exchange as discussed in
  {numref}`sec:ocl_steps` is to create memory objects (step 6).  i.e.,
  allocate buffers (regions) in the global memory for the kernels to
  use. There are two main ways to allocate global memory buffers:
  - **Allocation by XRT**:

    This is the method employed in the vector addition example in {numref}`sec:ocl_steps`:
    ```c++
    // Step 6: First part
    // These commands will allocate memory on the Device. The cl::Buffer objects can
    // be used to reference the memory locations on the device.
    OCL_CHECK(err, cl::Buffer buffer_a(context, CL_MEM_ALLOC_HOST_PTR|CL_MEM_READ_ONLY, 
      size_in_bytes, NULL, &err));
    OCL_CHECK(err, cl::Buffer buffer_b(context, CL_MEM_ALLOC_HOST_PTR|CL_MEM_READ_ONLY, 
      size_in_bytes, NULL, &err));
    OCL_CHECK(err, cl::Buffer buffer_result(context, CL_MEM_ALLOC_HOST_PTR|CL_MEM_WRITE_ONLY, 
      size_in_bytes, NULL, &err));

    ...

    // Step 6: Second part
    // We then need to map our OpenCL buffers to get the pointers
    int* ptr_a;
    int* ptr_b;
    int* ptr_result;
    OCL_CHECK(err, ptr_a = (int*) q.enqueueMapBuffer(buffer_a,
      CL_TRUE, CL_MAP_WRITE, 0, size_in_bytes, NULL, NULL, &err));
    OCL_CHECK(err, ptr_b = (int*) q.enqueueMapBuffer(buffer_b, 
      CL_TRUE, CL_MAP_WRITE, 0, size_in_bytes, NULL, NULL, &err));
    OCL_CHECK(err, ptr_result = (int*) q.enqueueMapBuffer(buffer_result, 
      CL_TRUE, CL_MAP_READ, 0, size_in_bytes, NULL, NULL, &err));
    ```
       - First, instantiate a `cl::Buffer` memory object using the
         `CL_MEM_ALLOC_HOST_PTR` flag to let XRT allocate the buffer
         in the global memory and the corresponding region in the host
         memory that is mapped to the global memory buffer, taking
         care of all alignment requirements.
       - Then, put an `enqueueMapBuffer` command to the command queue
         to obtain the handle (pointer) to the mapped region in the
         host memory.
       - In the example above, `buffer_a`, `buffer_b`, and
         `buffer_result` are pointers to the respective buffers in the
         global memory while `ptr_a`, `ptr_b`, and `ptr_result` are
         pointers to the corresponding mapped regions in the host
         memory.

  - **Allocation by host**:
     
     This method can be explained using this
     [example](https://github.com/Xilinx/Vitis_Accel_Examples/blob/2023.1/host/device_only_buffer/src/host.cpp)
     from the [Vitis repository](https://github.com/Xilinx/Vitis_Accel_Examples/tree/2023.1):
    ```c++
    // Use this allocator if user wish to create Buffer/Memory Object with 
    // CL_MEM_USE_HOST_PTR to align user buffer to the page boundary. 
    // It will ensure that user buffer will be used when user create
    // Buffer/Mem Object with CL_MEM_USE_HOST_PTR.
    template <typename T>
      struct aligned_allocator {
      using value_type = T;

      aligned_allocator() {}

      aligned_allocator(const aligned_allocator&) {}

      template <typename U>
      aligned_allocator(const aligned_allocator<U>&) {}

      T* allocate(std::size_t num) {
        void* ptr = nullptr;
        if (posix_memalign(&ptr, 4096, num * sizeof(T))) throw std::bad_alloc();
        return reinterpret_cast<T*>(ptr);
      }
      
      void deallocate(T* p, std::size_t num) {
        free(p);
      }
    };

        
    // allocate memory on host for input and output matrices
    std::vector<int, aligned_allocator<int> > A(array_size);
    std::vector<int, aligned_allocator<int> > B(array_size);
    std::vector<int, aligned_allocator<int> > C(array_size, 1);
    std::vector<int, aligned_allocator<int> > hw_results(array_size, 0);
    
    ...
    
    // Allocate Buffer in Global Memory
    OCL_CHECK(err, cl::Buffer buffer_a(context, CL_MEM_USE_HOST_PTR|CL_MEM_READ_ONLY, 
      size_in_bytes, A.data(), &err));
    OCL_CHECK(err, cl::Buffer buffer_b(context, CL_MEM_USE_HOST_PTR|CL_MEM_READ_ONLY, 
      size_in_bytes, B.data(), &err));
    OCL_CHECK(err, cl::Buffer buffer_c(context, CL_MEM_USE_HOST_PTR|CL_MEM_READ_ONLY, 
      size_in_bytes, C.data(), &err));
    OCL_CHECK(err, cl::Buffer buffer_d(context, CL_MEM_USE_HOST_PTR|CL_MEM_WRITE_ONLY,
      size_in_bytes, hw_results.data(), &err));
    ```
     - First, instantiate a buffer in the host memory with page
       alignment (e.g., use the `align allocator` template as shown
       above). 
     - Then, instantiate a `cl::Buffer` memory object using the
        `CL_MEM_USE_HOST_PTR` flag to tell XRT to allocate the buffer
        in the global memory and map provided host pointer to the
        global memory buffer.

* After the memory objects are created, we can transfer data between
  the host and global memory by putting `enqueueMigrateMemObjects`
  commands in the command queue to synchronize the contents of the
  mapped buffers in the host and global memory as done in the example
  in {numref}`sec:ocl_steps`:
  ```c++
  // Step 7: Transfer data from host to kernel 
  OCL_CHECK(err, err = q.enqueueMigrateMemObjects({buffer_a, buffer_b}, 0 
    /* 0 means from host*/));

  ... 
  
  // Step 9:
  // The result of the previous kernel execution will need to be retrieved in
  // order to view the results. This call will transfer the data from FPGA to
  // source_results vector
  OCL_CHECK(err, q.enqueueMigrateMemObjects({buffer_result}, 
    CL_MIGRATE_MEM_OBJECT_HOST));
  ```
    - The flag `CL_MIGRATE_MEM_OBJECT_HOST` indicates the direction of
      migration is from global memory to host memory. If the flag
      value is `0`, then the default direction of migration is from
      host memory to global memory.

## Pipelining Data Transfer and Kernel Execution

* Consider again the vector addition example in
  {numref}`sec:ocl_steps`, the command queue is set up to operate in
  the in-order execution mode by default in the code line
  ```c++
  OCL_CHECK(err, q = cl::CommandQueue(context, device, CL_QUEUE_PROFILING_ENABLE, &err));
  ```
  Under the in-order execution mode, the two writes to `buffer_a` and
  `buffer_b` from the host memory to the global memory have to
  complete, one after the other, before the kernel can be
  executed. In turn, the execution of the kernel has to complete
  before the result can be migrated from `buffer_result` in the
  global memory back to the host memory. Thus, if we break down the
  addition of two long vectors into processing shorter blocks by calling the
  kernel multiple times in the host application, the following
  execution timing diagram will result:
  ```{figure} ../figs/in-order.png
  ---
  name: in-order
  alt: Timing diagram of in-order execution of commands in the command queue
  width: 800px
  align: center
  ---
  Timing diagram of in-order execution of commands in the command queue (figure taken from
  {cite}`ug1393`)
  ```
  where `Wa0` and `Wb0` stand for writing data from host memory to
  `buffer_a` and `buffer_b` in global memory, and `Rc0` stands for
  reading data from `buffer_result` in global memory back to host
  memory.  We see from the timing diagram that there are large idling
  gaps in using the AXI interconnect for data transfer between the
  host and the kernel as well as large idling gaps in executing the
  kernel in the PL. These idling gaps are due to the latency of data
  transfer and that of the kernel. They lower the throughput achieved
  by the overall implementation, despite the kernel implementation may
  have been optimized.

* To increase the throughput of the overall implementation, we can
  overlap the data transfer between the host and the kernel and the
  execution of the kernel across multiple blocks. In order to achieve
  this form of *command-level* pipelining, we need to:
  1. make the command queue operate in the out-of-order execution mode,
  2. use more buffers, e.g., PIPOs, to store data of multiple blocks, and
  3. synchronize the sequence of transfer of data from host to
     global memory, kernel execution, and transfer of results back
     from global to host memory for each block.

  To see how this is done, consider the
  [example](https://github.com/Xilinx/Vitis_Accel_Examples/tree/2023.1/host/overlap)
  from Vitis' repository below:
  ```c++
  #include "xcl2.hpp"

  #include <algorithm>
  #include <cstdio>
  #include <random>
  #include <vector>

  using std::default_random_engine;
  using std::generate;
  using std::uniform_int_distribution;
  using std::vector;

  const int ARRAY_SIZE = 1 << 14;

  int gen_random() {
    static default_random_engine e;
    static uniform_int_distribution<int> dist(0, 100);

    return dist(e);
  }

  // An event callback function that prints the operations performed by the OpenCL
  // runtime.
  void event_cb(cl_event event1, cl_int cmd_status, void* data) {
    cl_int err;
    cl_command_type command;
    cl::Event event(event1, true);
    OCL_CHECK(err, err = event.getInfo(CL_EVENT_COMMAND_TYPE, &command));
    cl_int status;
    OCL_CHECK(err, err = event.getInfo(CL_EVENT_COMMAND_EXECUTION_STATUS, &status));
    const char* command_str;
    const char* status_str;
    switch (command) {
      case CL_COMMAND_READ_BUFFER:
        command_str = "buffer read";
        break;
      case CL_COMMAND_WRITE_BUFFER:
        command_str = "buffer write";
        break;
      case CL_COMMAND_NDRANGE_KERNEL:
        command_str = "kernel";
        break;
      case CL_COMMAND_MAP_BUFFER:
        command_str = "kernel";
        break;
      case CL_COMMAND_COPY_BUFFER:
        command_str = "kernel";
        break;
      case CL_COMMAND_MIGRATE_MEM_OBJECTS:
        command_str = "buffer migrate";
        break;
        default:
          command_str = "unknown";
    }
    switch (status) {
      case CL_QUEUED:
        status_str = "Queued";
        break;
      case CL_SUBMITTED:
        status_str = "Submitted";
        break;
      case CL_RUNNING:
        status_str = "Executing";
        break;
      case CL_COMPLETE:
        status_str = "Completed";
        break;
    }
    printf("[%s]: %s %s\n", reinterpret_cast<char*>(data), status_str, command_str);
    fflush(stdout);
  }

  // Sets the callback for a particular event
  void set_callback(cl::Event event, const char* queue_name) {
    cl_int err;
    OCL_CHECK(err, err = event.setCallback(CL_COMPLETE, event_cb, (void*)queue_name));
  }

  int main(int argc, char** argv) {
    if (argc != 2) {
      std::cout << "Usage: " << argv[0] << " <XCLBIN File>" << std::endl;
      return EXIT_FAILURE;
    }

    auto binaryFile = argv[1];
    cl_int err;
    cl::CommandQueue q;
    cl::Context context;
    cl::Kernel krnl_vadd;

    // OPENCL HOST CODE AREA START
    // get_xil_devices() is a utility API which will find the xilinx
    // platforms and will return list of devices connected to Xilinx platform
    std::cout << "Creating Context..." << std::endl;
    auto devices = xcl::get_xil_devices();

    // read_binary_file() is a utility API which will load the binaryFile
    // and will return the pointer to file buffer.
    auto fileBuf = xcl::read_binary_file(binaryFile);
    cl::Program::Binaries bins{{fileBuf.data(), fileBuf.size()}};
    bool valid_device = false;
    for (unsigned int i = 0; i < devices.size(); i++) {
      auto device = devices[i];
      // Creating Context and Command Queue for selected Device
      OCL_CHECK(err, context = cl::Context(device, nullptr, nullptr, nullptr, &err));
      // This example will use an out of order command queue. The default command
      // queue created by cl::CommandQueue is an inorder command queue.
      OCL_CHECK(err, q = cl::CommandQueue(context, device, 
        CL_QUEUE_OUT_OF_ORDER_EXEC_MODE_ENABLE, &err));

      std::cout << "Trying to program device[" << i << "]: " << 
        device.getInfo<CL_DEVICE_NAME>() << std::endl;
      cl::Program program(context, {device}, bins, nullptr, &err);
      if (err != CL_SUCCESS) {
        std::cout << "Failed to program device[" << i << "] with xclbin file!\n";
      } else {
        std::cout << "Device[" << i << "]: program successful!\n";
        OCL_CHECK(err, krnl_vadd = cl::Kernel(program, "vadd", &err));
        valid_device = true;
        break; // we break because we found a valid device
      }
    }
    if (!valid_device) {
      std::cout << "Failed to program any device found, exit!\n";
      exit(EXIT_FAILURE);
    }

    // We will break down our problem into multiple iterations. Each iteration
    // will perform computation on a subset of the entire data-set.
    size_t elements_per_iteration = 2048;
    size_t bytes_per_iteration = elements_per_iteration * sizeof(int);
    size_t num_iterations = ARRAY_SIZE / elements_per_iteration;

    // Allocate memory on the host and fill with random data.
    vector<int, aligned_allocator<int> > A(ARRAY_SIZE);
    vector<int, aligned_allocator<int> > B(ARRAY_SIZE);
    generate(begin(A), end(A), gen_random);
    generate(begin(B), end(B), gen_random);
    vector<int, aligned_allocator<int> > device_result(ARRAY_SIZE);

    // THIS PAIR OF EVENTS WILL BE USED TO TRACK WHEN A KERNEL IS FINISHED WITH
    // THE INPUT BUFFERS. ONCE THE KERNEL IS FINISHED PROCESSING THE DATA, A NEW
    // SET OF ELEMENTS WILL BE WRITTEN INTO THE BUFFER.
    vector<cl::Event> kernel_events(2);
    vector<cl::Event> read_events(2);
    cl::Buffer buffer_a[2], buffer_b[2], buffer_c[2];

    for (size_t iteration_idx = 0; iteration_idx < num_iterations; iteration_idx++) {
      int flag = iteration_idx % 2;

      if (iteration_idx >= 2) {
        OCL_CHECK(err, err = read_events[flag].wait());
      }

      // Allocate Buffer in Global Memory
      // Buffers are allocated using CL_MEM_USE_HOST_PTR for efficient memory and
      // Device-to-host communication
      std::cout << "Creating Buffers..." << std::endl;
      OCL_CHECK(err, buffer_a[flag] = cl::Buffer(context, CL_MEM_READ_ONLY|CL_MEM_USE_HOST_PTR, 
        bytes_per_iteration, &A[iteration_idx * elements_per_iteration], &err));
      OCL_CHECK(err, buffer_b[flag] = cl::Buffer(context, CL_MEM_READ_ONLY|CL_MEM_USE_HOST_PTR, 
        bytes_per_iteration, &B[iteration_idx * elements_per_iteration], &err));
      OCL_CHECK(err, buffer_c[flag] = cl::Buffer(context, CL_MEM_WRITE_ONLY|CL_MEM_USE_HOST_PTR, 
        bytes_per_iteration, &device_result[iteration_idx * elements_per_iteration], &err));

      vector<cl::Event> write_event(1);

      OCL_CHECK(err, err = krnl_vadd.setArg(0, buffer_c[flag]));
      OCL_CHECK(err, err = krnl_vadd.setArg(1, buffer_a[flag]));
      OCL_CHECK(err, err = krnl_vadd.setArg(2, buffer_b[flag]));
      OCL_CHECK(err, err = krnl_vadd.setArg(3, int(elements_per_iteration)));

      // Copy input data to device global memory
      std::cout << "Copying data (Host to Device)..." << std::endl;
      // Because we are passing the write_event, it returns an event object
      // that identifies this particular command and can be used to query
      // or queue a wait for this particular command to complete.
      OCL_CHECK(err, err = q.enqueueMigrateMemObjects({buffer_a[flag], buffer_b[flag]},
        0 /*0 means from host*/, nullptr, &write_event[0]));
      set_callback(write_event[0], "ooo_queue");

      printf("Enqueueing NDRange kernel.\n");
      // This event needs to wait for the write buffer operations to complete
      // before executing. We are sending the write_events into its wait list to
      // ensure that the order of operations is correct.
      // Launch the Kernel
      std::vector<cl::Event> waitList;
      waitList.push_back(write_event[0]);
      OCL_CHECK(err, err = q.enqueueNDRangeKernel(krnl_vadd, 0, 1, 1, 
        &waitList, &kernel_events[flag]));
      set_callback(kernel_events[flag], "ooo_queue");

      // Copy Result from Device Global Memory to Host Local Memory
      std::cout << "Getting Results (Device to Host)..." << std::endl;
      std::vector<cl::Event> eventList;
      eventList.push_back(kernel_events[flag]);
      // This operation only needs to wait for the kernel call. This call will
      // potentially overlap the next kernel call as well as the next read
      // operations
      OCL_CHECK(err, err = q.enqueueMigrateMemObjects({buffer_c[flag]}, 
        CL_MIGRATE_MEM_OBJECT_HOST, &eventList, &read_events[flag]));
      set_callback(read_events[flag], "ooo_queue");
    }

    // Wait for all of the OpenCL operations to complete
    printf("Waiting...\n");
    OCL_CHECK(err, err = q.flush());
    OCL_CHECK(err, err = q.finish());
    // OPENCL HOST CODE AREA ENDS
    bool match = true;
    // Verify the results
    for (int i = 0; i < ARRAY_SIZE; i++) {
      int host_result = A[i] + B[i];
      if (device_result[i] != host_result) {
        printf("Error: Result mismatch:\n");
        printf("i = %d CPU result = %d Device result = %d\n", i, host_result, device_result[i]);
        match = false;
        break;
      }
    }

    printf("TEST %s\n", (match ? "PASSED" : "FAILED"));
    return (match ? EXIT_SUCCESS : EXIT_FAILURE);
  }
  ```
  - Vectors of length $2^{14} = 16384$ are broken down into blocks of
      length $2048$ for the vector addition kernel `krnl_vadd` to
      operate on.
  - PIPO buffers, `buffer_a[2]`, `buffer_b[2]`, and `buffer_c[2]`, are
    instantiated to store the input and output blocks of data for
    `krnl_vadd` in each iteration of calling `krnl_vadd` to process a
    block. The "allocation by host" approach discussed in
    {numref}`sec:data_transfer` is employed to map an alternating
    component of the PIPO buffer to the location of the corresponding
    block of data in the host memory.
  - An out-of-order command queue is instantiated using the
    `CL_QUEUE_OUT_OF_ORDER_EXEC_MODE_ENABLE, &err));` flag.
   
* The execution of the example code follows the timing diagram below:
  ```{figure} ../figs/out-of-order.png
  ---
  name: out-of-order
  alt: Timing diagram of overlapped execution of commands in the command queue
  width: 800px
  align: center
  ---
  Timing diagram of overlapped execution of commands in the command queue (figure taken from
  {cite}`ug1393`)
  ```
  where `Wak` and `Wbk` stand for writing data from host memory to the
  PIPO buffer components `buffer_a[k]` and `buffer_b[k]` in global
  memory, and `Rck` stands for reading data from `buffer_result[k]` in
  global memory back to host memory, for `k=0,1`.

  - We see that the data transfer and kernel execution overlap in such
    a way that the idling gaps between successive executions of the
    kernel are significantly shortened, resulting in an increase in
    the throughput of the overall implementation.
  - Each arrow-linked sequence of operations (issued as commands to
    the command queue) in the timing diagram above indicates that the
    operations must be performed in order, i.e., the current operation
    must be completed before the next operation in the sequence can
    start.
  - The synchronization of an ordered sequence of commands is achieved
    by the use of vectors `std::vector<cl::Event>` of class
    `cl::Event` objects, `kernel_events`, `read_events`, and
    `write_event`. For example,
    1. The command of transferring data in the host memory to the PIPO
       buffers in the global memory is first issued by
       ```c++
       err = q.enqueueMigrateMemObjects({buffer_a[flag], buffer_b[flag]}, 0,
         nullptr, &write_event[0]);
       set_callback(write_event[0], "ooo_queue");
       ``` 
       where the event `write_event[0]` is associated with this
       command, and the `set_callback()` function is simply a wrapper
       to call the class method `setCallback()` of `write_event[0]` to
       set up the callback function handler `event_cb()` to print out
       information about the execution of the command when it
       completes, as set by the `CL_COMPLETE` flag in the first
       argument of `setCallback()`.
    2. The command of execution of `krnl_vadd` is then issued by
       ```c++
       std::vector<cl::Event> waitList;
       waitList.push_back(write_event[0]);
       err = q.enqueueNDRangeKernel(krnl_vadd, 0, 1, 1, &waitList, 
         &kernel_events[flag]);
       set_callback(kernel_events[flag], "ooo_queue");
       ```
        where the event `kernel_events[flag]` is associated with this
        command, and the command should wait for the (one) event
        `write_event[0]` in `waitList` to complete before it starts.
     3. The command of transferring results back from the PIPO buffer
        in the global memory is finally issued by
        ```c++
        std::vector<cl::Event> eventList;
        eventList.push_back(kernel_events[flag]);
        err = q.enqueueMigrateMemObjects({buffer_c[flag]}, 
          CL_MIGRATE_MEM_OBJECT_HOST, &eventList, &read_events[flag]);
        set_callback(read_events[flag], "ooo_queue");
        ```
        where the event `read_events[flag]` is associated with this
        command, and the command should wait for the event `kernel_events[flag]`
        in `eventList` to complete before it starts.
     4. Then, in the next iteration that use the same PIPO component
        buffers as indicated by the value `flag`, we wait for the command in 3. to
        complete using
        ```c++
        err = read_events[flag].wait();
        ```
        before restarting the sequence of operations.
  
 

  
