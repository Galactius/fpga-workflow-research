# Technical Information

## Table of Contents

1. [**Basics**](#basics) 
2. [**Deploying an AWS F1 Instance**](#deploying-an-aws-f1-instance) 
5. [**Useful Links and Sources**](#useful-links-and-sources)

## Basics 

An FPGA or *Field-Programmable Gate Array*, is essentially a processor with integrated reprogrammable logic gates that make running one specific program efficient, as when fully compiled, a program will be baked into the FPGA's physical logic. This means that while an FPGA can only run one specific application at a time, it can be reprogrammed to run any different application. This is in direct contrast to an ASIC or *Application-Specific Integrated Circuit*, which has better performance overall, but cannot be reprogrammed to run different applications, its hardware must be completely tailored to run one singular program. 

While hobbyist or more budget-friendly physical FPGA and virtual FPGA solutions exist, developing an ASIC can be costly for non-business ventures and was thus not considered for this project. In my research, I made use of *Amazon Web Services'* (AWS) FPGA development platform and workflow, which uses AMD's Xilinx platform to develop virtual FPGA's and deploy them to a cloud-based FPGA through AWS's F1 instances. 

## Deploying an AWS F1 Instance

After getting my account setup through my supervisor's parent account, I spun up one "free-tier" instance in order to become acquainted with AWS and the AWS CLI API. During this first phase I also started setting up SSH keys, my process for file transfer using Filezilla, and how to navigate the AWS dashboard. Once I was comfortable using the CLI and AWS in my terminal and could confirm that my SSH keys were setup properly to authenticate as needed, I created a new, more powerful virtual machine to actually start building FPGA binaries, etc. 

The workflow for developing FPGA's involves:
 
1. Starting an AWS instance with the AWS FPGA Dev. Image from the AWS Marketplace (this does not need to be the special "F1 FPGA" instance, I selected a cheaper, general purpose VM to save cost). 
2. Binding the correct SSH keys from the host (local) machine to the remote AWS instance.  
3.  Configuring the AWS CLI API through both the AWS EC2 dashboard and entering information the instance's terminal using "aws-cli ...", as well as setting up an S3 bucket. 
4. Cloning the AWS-FPGA git to the AWS instance's home directory *Optional: run `screen`*. 
5. Navigating to the main directory inside the cloned AWS-FPGA git and enter the following commands:
 
```
    source vitis_setup.sh
```
 6. Allow the vitis setup script to run to completion. Run `screen`if you have not already. Ensure that only one TARGET is entered. Both `sw_emu` and `hw_emu` will run emulations of the binaries for debugging purposes, while the `hw` TARGET will build the Host application and FPGA binaries. 
To build or emulate the FPGA binaries run the following commands:

``` 
    cd $VITIS_DIR/directory/containing/fpga/source_files
    make clean
    make TARGET=[sw_emu, hw_emu, or hw] DEVICE=$AWS_PLATFORM all 
```
7. Upon successful emulation and/or compilation, the Host application (usually has no extension) and the FPGA binary files (.xclbin file) will be generated, keep track of these two files. 
8. The final step is to generate the Amazon FPGA Image, do so by running the following commands: 
 
```
    cd $VITIS_DIR/tools
    ./create_vitis_afi.sh -xclbin=/path/to/.xclbinfile -o=vadd -s3_bucket=unique_s3_bucket_name -s3_dcp_key=dcpFolderName -s3_logs_key=s3LogsFolderName
```
9. Transferring the compiled executable (my executable was found in the corresponding example folder) and .awsxclbin file from the tools directory. *Note: AFI Generation executes in the background on a different machine within the AWS network and continues even if the instance used to generate the AFI request is offline, the AWS-CLI must be invoked to ascertain the status of the AFI generation.* 

## Useful Links and Sources
- AWS-FPGA Github Repo: [https://github.com/aws/aws-fpga](https://github.com/aws/aws-fpga) (Documentation for AWS FPGA F1 instances)
- AMD Xilinx Documentation: [https://docs.xilinx.com/home](https://docs.xilinx.com/home) (Xilinx-specific documentation, includes links for Vitis & Vivado)
- AMD Xilinx Wiki: [https://xilinx-wiki.atlassian.net/wiki/spaces/A/overview](https://xilinx-wiki.atlassian.net/wiki/spaces/A/overview]) (General Xilinx FPGA information)
- AMD Xilinx University Program Vitis Tutorial: [https://xilinx.github.io/xup_compute_acceleration/index.html](https://xilinx.github.io/xup_compute_acceleration/index.html) (Tutorials on using Xilinx's Vitis tools and other general information)
- HPC Challenge FPGA Applications Github: [https://github.com/pc2/HPCC_FPGA](https://github.com/pc2/HPCC_FPGA) (FPGA Benchmark suite using HPC challenge programs) 


