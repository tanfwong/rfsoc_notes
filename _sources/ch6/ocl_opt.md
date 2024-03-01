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
