# Running C/C++ Application on an AWS F1 FPGA Instance with Vitis
There are three steps for accelerating your application on an Amazon EC2 FPGA instance:
1. Build the host application, and the Xilinx FPGA binary 
2. Create an AFI (Amazon FPGA Image) 
3. Run the FPGA accelerated application on AWS FPGA F1 instances

A simple "Hello World" Vitis example to get you started.

# Hello World Vitis Example
This example uses the load/compute/store coding style which is generally
the most efficient for implementing kernels using HLS. The load and store
functions are responsible for moving data in and out of the kernel as
efficiently as possible. The core functionality is decomposed across one
of more compute functions. Whenever possible, the compute function should
pass data through HLS streams and should contain a single set of nested loops.

HLS stream objects are used to pass data between producer and consumer
functions. Stream read and write operations have a blocking behavior which
allows consumers and producers to synchronize with each other automatically.

The dataflow pragma instructs the compiler to enable task-level pipelining.
This is required for to load/compute/store functions to execute in a parallel
and pipelined manner.

The kernel operates on vectors of NUM_WORDS integers modeled using the hls::vector
data type. This datatype provides intuitive support for parallelism and
fits well the vector-add computation. The vector length is set to NUM_WORDS
since NUM_WORDS integers amount to a total of 64 bytes, which is the maximum size of
a kernel port. It is a good practice to match the compute bandwidth to the I/O
bandwidth. Here the kernel loads, computes and stores NUM_WORDS integer values per
clock cycle and is implemented as below:

<img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/f24dbe41-45bd-46ff-a7f7-b44a68200c05" width=70%>

Hello world sources are [host.cpp](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/hello_world/src/host.cpp) and [vadd.cpp](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/hello_world/src/vadd.cpp).

