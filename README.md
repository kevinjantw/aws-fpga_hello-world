# Running C/C++ Application on an AWS F1 FPGA Instance with Vitis
There are three steps for accelerating your application on an Amazon EC2 FPGA instance: 
* Build the host application, and the Xilinx FPGA binary
* Create an AFI (Amazon FPGA Image)
* Run the FPGA accelerated application on AWS FPGA F1 instance
   
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

Hello world sources: [host.cpp](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/hello_world/src/host.cpp) and [vadd.cpp](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/hello_world/src/vadd.cpp).

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
  (13) SSH connection to EC2 IP with pem file and username `centos`, here is a MobaXterm example

  <img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/248380d6-7f5c-403c-b478-819694203cf1" width=70%>
  
  (14) Configure EC2 and S3 connection (fill access key in downloaded csv file and your region)
  ```console
  $ aws configure
  AWS Access Key ID [None]: AKIA4***********MHKE
  AWS Secret Access Key [None]: hnu++***************************fO7tLW7o
  Default region name [None]: us-east-1
  Default output format [None]:
  ```
  (15) Test transferring data between EC2 and S3
  ```console
  $ echo 'test ec2 to s3c' > test-ec2-s3storage.txt
  $ aws s3 cp test-ec2-s3storage.txt s3://test-ec2-s3storage/test-ec2-s3storage.txt
  upload: ./test-ec2-s3storage.txt to s3://test-ec2-s3storage/test-ec2-s3storage.txt
  ```
  <img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/85611197-6d00-4d1a-b85b-cb87c50713a1" width=60%>
  
* Request access to Amazon EC2 F1 instances  
  (01) Open the `Service quota increase` [form](http://aws.amazon.com/contact-us/ec2-request)  
  (02) Submit a Service limit increase for EC2 Instances   
  (03) Select the region where you want to access F1 instances: US East (N.Virginia), US West (Oregon) or EU (Ireland)  
  (04) Select the instance type, either `f1.2xlarge` or `f1.16xlarge`  
  (05) Set the new limit value to 1 or more  
  (06) Fill the rest of the form as appropriate and click `Submit`

* Reference offical document  
  [Quick Start Guide to Accelerating your C/C++ application on an AWS F1 FPGA Instance with Vitis](https://github.com/aws/aws-fpga/blob/master/Vitis/README.md#build-the-host-application-and-xilinx-fpga-binary)   
  
# Build the host application and Xilinx FPGA binary
(01) Subscribe a [FPGA Developer AMI v1.12.2](https://aws.amazon.com/marketplace/pp/prodview-gimv3gqbpe57k?sr=0-1&ref_=beagle&applicationId=AWSMPContessa) and use a performance recommended `z1d.2xlarge` instance type
 
 <img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/5befcd47-f88f-4a79-ab3d-1b84a35aa45d" width=70%>

(02) SSH connection to the running EC2 `z1d.2xlarge` instance, your EC2 public IP can be found in [EC2 Instances](https://console.aws.amazon.com/ec2/home#Instances)

(03) Clone this github repository and source the *vitis_setup.sh* script  
  * AWS Vitis Platform that contains the dynamic hardware that enables Vitis kernels to run on AWS F1 instances  
  * Valid platforms for shell_v04261818: AWS_PLATFORM_201920_3 (Default) AWS F1 Vitis platform  
  * Sets up the Xilinx Vitis example submodules  
  * Installs the required libraries and package dependencies  
  * Run environment checks to verify supported tool/lib versions  
  ```console
  $ git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
  $ cd $AWS_FPGA_REPO_DIR
  $ source vitis_setup.sh
  ```
  [vitis_setup.log](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/logs/vitis_setup.log)
  
(04) Software (SW) Emulation  
  For CPU-based (SW) emulation, both the host code and the FPGA binary code are compiled to run on an x86 processor. SW Emulation enables developers to iterate and refine 
  the algorithms through fast compilation. The iteration time is similar to software compile and run cycles on a CPU.
  ```console
  $ cd $VITIS_DIR/examples/xilinx/hello_world
  $ make clean
  $ make run TARGET=sw_emu DEVICE=$AWS_PLATFORM all
  ```
  [sw_emulation.log](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/logs/sw_emulation.log)
  
(05) Hardware (HW) Emulation  
  The Vitis hardware emulation flow enables the developer to check the correctness of the logic generated for the FPGA binary. This emulation flow invokes the hardware
  simulator in the Vitis environment to test the functionality of the code that will be executed on the FPGA Custom Logic.
  ```console
  $ cd $VITIS_DIR/examples/xilinx/hello_world
  $ make clean
  $ make run TARGET=hw_emu DEVICE=$AWS_PLATFORM all
  ```
  [hw_emulation.log](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/logs/hw_emulation.log)
  
(06) Generate Hardware (HW) Xilinx FPGA Binary   
  The Vitis system build flow enables the developer to build their host application as well as their Xilinx FPGA Binary.
  ```console
  $ cd $VITIS_DIR/examples/xilinx/hello_world
  $ make clean
  $ make TARGET=hw DEVICE=$AWS_PLATFORM all
  ```
  [hw.log](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/logs/hw.log)
  
# Test on AWS FPGA F1 instance
The *create_vitis_afi.sh* script is provided to facilitate AFI (Amazon FPGA Image) creation from a xclbin (Xilinx FPGA Binary)
* Takes in your Xilinx FPGA Binary *.xclbin file
* Calls *aws ec2 create_fpga_image* to generate an AFI under the hood
* Generates a <timestamp>_afi_id.txt which contains the identifiers for your AFI
* Creates an AWS FPGA Binary file with an *.awsxclbin extension that is composed of: Metadata and AGFI-ID

Before running *create_vitis_afi.sh*, you need to
* Confirm vadd.xclbin was generated
* Create s3 bucket and folders for *create_vitis_afi.sh* writing
* Configure access between s3 and ec2

Example on [S3 Management Console](https://s3.console.aws.amazon.com/)

<img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/a86b224e-c9d3-4cb1-93cd-c250d262c0b6" width=70%>

(01) Configure AWS access
```console
  $ aws configure
  AWS Access Key ID [None]: AKIA4***********MHKE
  AWS Secret Access Key [None]: hnu++***************************fO7tLW7o
  Default region name [None]: us-east-1
  Default output format [None]:
```
(02) Run *create_vitis_afi.sh* and generate awsxclbin
```console
$ $VITIS_DIR/tools/create_vitis_afi.sh  \
>  -xclbin=/home/centos/src/project_data/aws-fpga/Vitis/examples/xilinx_2021.2/hello_world/build_dir.hw.xilinx_aws-vu9p-f1_shell-v04261818_201920_3/vadd.xclbin  \
>  -o=/home/centos/vadd -s3_bucket=f1-s3 -s3_dcp_key=aws_xclbin -s3_logs_key=log
```
[awsxclbin.log](https://github.com/kevinjantw/aws-fpga_hello-world/blob/main/logs/awsxclbin.log)

(03) Get AFI ID  
The *_afi_id.txt file generated by the create_vitis_afi.sh, AGFI ID is a global ID that is used to refer to an AFI from within an F1 instance
```console
$ cat 23_11_23-034100_afi_id.txt
{
    "FpgaImageId": "afi-001d35de56eed82fd",
    "FpgaImageGlobalId": "agfi-0d56be399f577f45c"
}
```
(04) Wait until directory afi-001d35de56eed82fd and its logs are written to s3

<img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/169db288-7a51-4682-a38b-059688beb4a3" width=70%>  
<img src="(https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/f00a74f4-a679-494b-9edf-2e0bc3350655" width=70%>
<img src="https://github.com/kevinjantw/aws-fpga_hello-world/assets/11850122/3dd94c8e-b0f8-481a-89e5-5a5ea7d8bd5a" width=70%>

(05) Download State file in directory afi-001d35de56eed82fd and check written State is available
```shell
"State": {
        "Code": "available"
}
```
(06) Copy `~/vadd.awsxclbin` and `/home/centos/src/project_data/aws-fpga/Vitis/examples/xilinx_2021.2/hello_world/hello_world` to local or s3

(07) Terminate running `z1d.2xlarge` instance in [EC2 Instances](https://console.aws.amazon.com/ec2/home#Instances)

(08) Subscribe a [FPGA Developer AMI v1.12.2](https://aws.amazon.com/marketplace/pp/prodview-gimv3gqbpe57k?sr=0-1&ref_=beagle&applicationId=AWSMPContessa) and use a `f1.2xlarge` instance type

(09) Copy saved `vadd.awsxclbin` and `hello_world` to running `f1.2xlarge` instance

(10) To setup tools and runtime environment
```console
$ git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
$ cd $AWS_FPGA_REPO_DIR
$ source vitis_runtime_setup.sh
# Wait till the MPD service has initialized. Check systemctl status mpd
```

(11) Execute your Host Application
```console
$ chmod +x ./hello_world
$ ./hello_world ./vadd.awsxclbin
Found Platform
Platform Name: Xilinx
INFO: Reading ./vadd.awsxclbin
Loading: './vadd.awsxclbin'
Trying to program device[0]: xilinx_aws-vu9p-f1_shell-v04261818_201920_3
Device[0]: program successful!
TEST PASSED
```
