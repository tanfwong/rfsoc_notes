# OpenCL Steps

* Under the VAAD flow, an application program running on the PS host
  may use the OpenCL platform-layer and runtime APIs to perform the
  following sequence of steps in order to execute computing tasks in
  the DSP kernels (compute units) implemented in the PL (compute
  device):
  1. Query for available OpenCL platforms and PL devices. For our
     case, we look for the Xilinx platform and find all devices in
     it.
  2. Create a context for each device in the Xilinx platform.
  3. Create one or more command queues to execute commands on
     each device.
  4. Create and build a program by loading the PL bit-stream
     (`.xclbin` file) into each device.
  5. Select kernels to execute from the program and set up the
     selected kernels.
  6. Create memory objects for the selected kernels to operate on.
  7. Enqueue memory commands into the command queue to transfer data
     from the host to global memory, if needed.
  8. Enqueue kernel execution commands into the command queue for
     execution.
  9. Enqueue commands into the command queue to transfer data back to
     the host, if needed.
  10. Unmap memory objects and stop the command queues. 

* To see how the steps above are carried out in a host application
  program, consider [Vitis' simple vector addition
  example](https://github.com/Xilinx/Vitis_Accel_Examples/tree/2023.1/cpp_kernels/simple_vadd)
  used in Lab 1:

  ```c++
  #define OCL_CHECK(error, call)                                                               \
    call;                                                                                      \
    if (error != CL_SUCCESS) {                                                                 \
      printf("%s:%d Error calling " #call ", error code is: %d\n", __FILE__, __LINE__, error); \
      exit(EXIT_FAILURE);                                                                      \
    }

  #include "vadd.h"
  #include <fstream>
  #include <iostream>
  #include <stdlib.h>

  static const int DATA_SIZE = 4096;

  static const std::string error_message =
    "Error: Result mismatch:\n"
    "i = %d CPU result = %d Device result = %d\n";

  int main(int argc, char* argv[]) {
    // TARGET_DEVICE macro needs to be passed from gcc command line
    if (argc != 2) {
      std::cout << "Usage: " << argv[0] << " <xclbin>" << std::endl;
      return EXIT_FAILURE;
    }

    std::string xclbinFilename = argv[1];

    // Compute the size of array in bytes
    size_t size_in_bytes = DATA_SIZE * sizeof(int);

    std::vector<cl::Device> devices;
    cl_int err;
    cl::Context context;
    cl::CommandQueue q;
    cl::Kernel krnl_vector_add;
    cl::Program program;
    std::vector<cl::Platform> platforms;
    bool found_device = false;

    // Step 1: 
    // Traversing all Platforms To find Xilinx Platform and targeted
    // Device in Xilinx Platform
    cl::Platform::get(&platforms);
    for (size_t i = 0; (i < platforms.size()) & (found_device == false); i++) {
      cl::Platform platform = platforms[i];
      std::string platformName = platform.getInfo<CL_PLATFORM_NAME>();
      if (platformName == "Xilinx") {
        devices.clear();
        platform.getDevices(CL_DEVICE_TYPE_ACCELERATOR, &devices);
        if (devices.size()) {
          found_device = true;
          break;
        }
      }
    }
    if (found_device == false) {
      std::cout << "Error: Unable to find Target Device " << std::endl;
      return EXIT_FAILURE;
    }

    // Load .xclbin into memory
    std::cout << "INFO: Reading " << xclbinFilename << std::endl;
    FILE* fp;
    if ((fp = fopen(xclbinFilename.c_str(), "r")) == nullptr) {
      printf("ERROR: %s xclbin not available please build\n", xclbinFilename.c_str());
      exit(EXIT_FAILURE);
    }
    std::cout << "Loading: '" << xclbinFilename << "'\n";
    std::ifstream bin_file(xclbinFilename, std::ifstream::binary);
    bin_file.seekg(0, bin_file.end);
    unsigned nb = bin_file.tellg();
    bin_file.seekg(0, bin_file.beg);
    char* buf = new char[nb];
    bin_file.read(buf, nb);

    // Going through all devices in the Xilinx platform
    cl::Program::Binaries bins;
    bins.push_back({buf, nb});
    bool valid_device = false;
    
    for (unsigned int i = 0; i < devices.size(); i++) {
      auto device = devices[i];
      // Step 2:
      // Creat Context for selected Device
      OCL_CHECK(err, context = cl::Context(device, nullptr, nullptr, nullptr, &err));
      // Step 3:
      // Creat Command Queue for selected Device
      OCL_CHECK(err, q = cl::CommandQueue(context, device, CL_QUEUE_PROFILING_ENABLE, &err));
      // Step 4:
      // Creat Program and load PL bit-stream to selected Device
      std::cout << "Trying to program device[" << i << "]: " << device.getInfo<CL_DEVICE_NAME>() << std::endl;
      cl::Program program(context, {device}, bins, nullptr, &err);
      if (err != CL_SUCCESS) {
        std::cout << "Failed to program device[" << i << "] with xclbin file!\n";
      } else {
        std::cout << "Device[" << i << "]: program successful!\n";
        OCL_CHECK(err, krnl_vector_add = cl::Kernel(program, "krnl_vadd", &err));
        valid_device = true;
        break; // we break because we found a valid device
      }
    }

    if (!valid_device) {
      std::cout << "Failed to program any device found, exit!\n";
      exit(EXIT_FAILURE);
    }

    // Step 6: First part
    // These commands will allocate memory on the Device. The cl::Buffer objects can
    // be used to reference the memory locations on the device.
    OCL_CHECK(err, cl::Buffer buffer_a(context, CL_MEM_READ_ONLY, size_in_bytes, NULL, &err));
    OCL_CHECK(err, cl::Buffer buffer_b(context, CL_MEM_READ_ONLY, size_in_bytes, NULL, &err));
    OCL_CHECK(err, cl::Buffer buffer_result(context, CL_MEM_WRITE_ONLY, size_in_bytes, NULL, &err));

    // Step 5:
    // Select kernels to execute and set the kernel Arguments
    int narg = 0;
    OCL_CHECK(err, err = krnl_vector_add.setArg(narg++, buffer_a));
    OCL_CHECK(err, err = krnl_vector_add.setArg(narg++, buffer_b));
    OCL_CHECK(err, err = krnl_vector_add.setArg(narg++, buffer_result));
    OCL_CHECK(err, err = krnl_vector_add.setArg(narg++, DATA_SIZE));


    // Step 6: Second part
    // We then need to map our OpenCL buffers to get the pointers
    int* ptr_a;
    int* ptr_b;
    int* ptr_result;
    OCL_CHECK(err,
              ptr_a = (int*)q.enqueueMapBuffer(buffer_a, CL_TRUE, CL_MAP_WRITE, 0, size_in_bytes, NULL, NULL, &err));
    OCL_CHECK(err,
              ptr_b = (int*)q.enqueueMapBuffer(buffer_b, CL_TRUE, CL_MAP_WRITE, 0, size_in_bytes, NULL, NULL, &err));
    OCL_CHECK(err, ptr_result = (int*)q.enqueueMapBuffer(buffer_result, CL_TRUE, CL_MAP_READ, 0, size_in_bytes, NULL,
                                                         NULL, &err));

    // Initialize the vectors used in the test
    for (int i = 0; i < DATA_SIZE; i++) {
      ptr_a[i] = rand() % DATA_SIZE;
      ptr_b[i] = rand() % DATA_SIZE;
    }

    // Step 7: Transfer data from host to kernel 
    OCL_CHECK(err, err = q.enqueueMigrateMemObjects({buffer_a, buffer_b}, 0 /* 0 means from host*/));

    // Step 8: Launch the Kernel
    OCL_CHECK(err, err = q.enqueueTask(krnl_vector_add));

    // Step 9:
    // The result of the previous kernel execution will need to be retrieved in
    // order to view the results. This call will transfer the data from FPGA to
    // source_results vector
    OCL_CHECK(err, q.enqueueMigrateMemObjects({buffer_result}, CL_MIGRATE_MEM_OBJECT_HOST));

    OCL_CHECK(err, q.finish());

    // Verify the result
    int match = 0;
    for (int i = 0; i < DATA_SIZE; i++) {
      int host_result = ptr_a[i] + ptr_b[i];
      if (ptr_result[i] != host_result) {
        printf(error_message.c_str(), i, host_result, ptr_result[i]);
        match = 1;
        break;
      }
    }

    // Step 10: Clean up
    OCL_CHECK(err, err = q.enqueueUnmapMemObject(buffer_a, ptr_a));
    OCL_CHECK(err, err = q.enqueueUnmapMemObject(buffer_b, ptr_b));
    OCL_CHECK(err, err = q.enqueueUnmapMemObject(buffer_result, ptr_result));
    OCL_CHECK(err, err = q.finish());

    std::cout << "TEST " << (match ? "FAILED" : "PASSED") << std::endl;
    return (match ? EXIT_FAILURE : EXIT_SUCCESS);
  }
  ```
  - Detailed usage information of the OpenCL APIs used in the example
     code above can be found in {cite}`opencl1.2`

* The PL kernel code for this example is shown below for reference:
  ```c++
  #include <stdint.h>
  #include <hls_stream.h>

  #define DATA_SIZE 4096

  // TRIPCOUNT identifier
  const int c_size = DATA_SIZE;

  static void load_input(uint32_t* in, hls::stream<uint32_t>& inStream, int size) {
    mem_rd: for (int i = 0; i < size; i++) {
  #pragma HLS LOOP_TRIPCOUNT min = c_size max = c_size
      inStream << in[i];
    }
  }

  static void compute_add(hls::stream<uint32_t>& in1_stream,
                          hls::stream<uint32_t>& in2_stream,
                          hls::stream<uint32_t>& out_stream,
                          int size) {
  // The kernel is operating with vector of NUM_WORDS integers. The + operator performs
  // an element-wise add, resulting in NUM_WORDS parallel additions.
  execute:
    for (int i = 0; i < size; i++) {
  #pragma HLS LOOP_TRIPCOUNT min = c_size max = c_size
      out_stream << (in1_stream.read() + in2_stream.read());
    }
  }

  static void store_result(uint32_t* out, hls::stream<uint32_t>& out_stream, int size) {
    mem_wr: for (int i = 0; i < size; i++) {
  #pragma HLS LOOP_TRIPCOUNT min = c_size max = c_size
      out[i] = out_stream.read();
    }
  }

  extern "C" {
  /*
    Vector Addition Kernel

    Arguments:
        in1  (input)  --> Input vector 1
        in2  (input)  --> Input vector 2
        out  (output) --> Output vector
        size (input)  --> Number of elements in vector
  */

  void krnl_vadd(uint32_t* in1, uint32_t* in2, uint32_t* out, int size) {
  #pragma HLS INTERFACE m_axi port = in1 bundle = gmem0
  #pragma HLS INTERFACE m_axi port = in2 bundle = gmem1
  #pragma HLS INTERFACE m_axi port = out bundle = gmem0

    static hls::stream<uint32_t> in1_stream("input_stream_1");
    static hls::stream<uint32_t> in2_stream("input_stream_2");
    static hls::stream<uint32_t> out_stream("output_stream");

  #pragma HLS dataflow
    // dataflow pragma instruct compiler to run following three APIs in parallel
    load_input(in1, in1_stream, size);
    load_input(in2, in2_stream, size);
    compute_add(in1_stream, in2_stream, out_stream, size);
    store_result(out, out_stream, size);
  }
  }
  ```
  ```{tip}
  In order for Vitis to properly build both the host code and kernel
  code in a system project, the top-level function of the kernel must
  be wrapped with the `extern "C"` linkage as shown in the example
  above. This is to avoid the Vitis' C++ compiler from adding argument
  information to the name of the top-level function used for linkage.
  ```
