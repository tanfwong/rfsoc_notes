# OpenCL Steps

* Under the VAAD flow, an application program running on the PS host
  may use the OpenCL platform-layer and runtime APIs to perform the
  following sequence of steps in order to execute computing tasks in
  the DSP kernels (compute units) implemented in the PL (compute
  device):
  1. Query for available OpenCL platforms and PL devices.
  2. Create a context for a PL device in a platform.
  3. Create and build a program by loading the PL bit-stream
     (`.xclbin` file) into the PL device in the context.
  4. Create a single out-of-order command queue to execute commands on
     the PL device.
  5. Select kernels to execute from the program and set up the
     selected kernels.
  6. Create memory objects for the selected kernels to operate on.
  7. Enqueue memory commands into the command queue to transfer data
     from the host to global memory, if needed.
  8. Enqueue kernel execution commands into the command queue for
     execution.
  9. Enqueue commands into the command queue to transfer data back to
     the host, if needed.
