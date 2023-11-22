# Running C/C++ Application on an AWS F1 FPGA Instance with Vitis
There are three steps for accelerating your application on an Amazon EC2 FPGA instance:
1. Build the host application, and the Xilinx FPGA binary 
2. Create an AFI (Amazon FPGA Image) 
3. Run the FPGA accelerated application on AWS FPGA F1 instance

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

# Prerequisites
* [Create an AWS account](https://aws.amazon.com/free/) [Currently EC2 t2.micro is free tier]
* Prepare SSH keys to access the Amazon EC2 instances  
  (01) Open the [EC2 Management Console](https://console.aws.amazon.com/ec2/home)  
  (02) In the left pane, select `Key Pairs` from the `Network & Security`  
  (03) Choose the `Create key pair` button  
  (04) Name your key pair and choose `Create key pair`  
  (05) Generate pem file with RSA type  
  (06) The downloaded pem file can be reused for each time staring an EC2 instance
* Create S3 credential  
  (01) Open the [EC2 Management Console](https://console.aws.amazon.com/ec2/home)  
  (02) Click on your `Name` on the right top bar  
  (03) Select `Security credentials`  
  (04) Click `Users` from the `Access management`  
  (05) Choose `Create user` button  
  (06) Add an `username`  
  (07) Select `Attach policies directly`  
  (08) Search and select `AmazonS3FullAccess` and `AmazonEC2FullAccess` permissions policies  
  (09) Click `Create user`  
  (10) Click your created `username` in `Users info page`  
  (11) Choose `Create access key` and select `Command Line Interface (CLI)`, then `Confirm`  
  (12) Click  and download csv file  
* Test transferring data between EC2 and S3  
  (01) Open the [S3 Management Console](https://s3.console.aws.amazon.com/)  
  (02) Click `Create bucket` and name `test-ec2-s3storage` with default settings  
  (03) Subscribe a [FPGA Developer AMI v1.12.2](https://aws.amazon.com/marketplace/pp/prodview-gimv3gqbpe57k?sr=0-1&ref_=beagle&applicationId=AWSMPContessa) and `Continue to Configuration`  
  (04) Click `Continue to Launch`  
  (05) Select `Launch through EC2` from `Choose Action`, then click `Launch`  
  (06) Give a `Name`  
  (07) Select `t2.micro` from Instance type  
  (08) Select `Key pair` which was generated before (a pem file)  
  (09) `Create security group` or `Select existing security group`(if created before)  
  (10) Click `Launch instance`  
  (11) Open the [EC2 Instances](https://console.aws.amazon.com/ec2/home#Instances)  
  (12) Get running EC2 instance IP from `Public IPv4`  
  (13) SSH connection to EC2 IP with pem file and username `centos`  
  (14) Configure EC2 and S3 connection (fill access key in downloaded csv file and your region)
  ```console
  $ aws configure
  AWS Access Key ID [None]: AKIA4ALHPDJIP7Y2MHKE
  AWS Secret Access Key [None]: hnu++p3ZUhT91CzSVQzO+/O0mCZOLpoufO7tLW7o
  Default region name [None]: us-east-1
  Default output format [None]:
  ```
  (15) Test transferring data between EC2 and S3
  ```console
  $ echo 'test ec2 to s3' > test-ec2-s3storage.txt
  $ aws s3 cp test-ec2-s3storage.txt s3://test-ec2-s3storage/test-ec2-s3storage.txt
  upload: ./test-ec2-s3storage.txt to s3://test-ec2-s3storage/test-ec2-s3storage.txt
  ```
  <img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/85611197-6d00-4d1a-b85b-cb87c50713a1" width=60%>

  # Build the host application, Xilinx FPGA binary and verify you are ready for AWS FPGA F1 instance
  * Run [FPGA Developer AMI v1.12.2](https://aws.amazon.com/marketplace/pp/prodview-gimv3gqbpe57k?sr=0-1&ref_=beagle&applicationId=AWSMPContessa) on a recommended `z1d.2xlarge` instance type
 
    <img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/5befcd47-f88f-4a79-ab3d-1b84a35aa45d" width=70%>

  * SSH connection to the running EC2 `z1d.2xlarge` instance, here is a MobaXterm example
    <img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/248380d6-7f5c-403c-b478-819694203cf1" width=70%>

    Your own EC2 IP can be founded in [EC2 Instances](https://console.aws.amazon.com/ec2/home#Instances)

  * Github and Environment 
