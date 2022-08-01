# Getting Started

## Table of Contents

1. [**Background Information**](#background-information) 
2. [**Setting up AWS**](#setting-up-aws)
3. [**Compiling and Deploying FPGA code to an F1 Instance**](#compiling-and-deploying-fpga-code-to-an-f1-instance) 
4. [**Useful Links and Sources**](#useful-links-and-sources)

## Background Information 

An FPGA or *Field-Programmable Gate Array*, is essentially a processor with integrated reprogrammable logic gates that make running one specific program efficient, as when fully compiled, a program will be baked into the FPGA's physical logic. This means that while an FPGA can only run one specific application at a time, it can be reprogrammed to run any different application. This is in direct contrast to an ASIC or *Application-Specific Integrated Circuit*, which has better performance overall, but cannot be reprogrammed to run different applications, its hardware must be completely tailored to run one singular program. 

While hobbyist or more budget-friendly physical FPGA and virtual FPGA solutions exist, developing an ASIC can be costly for non-business ventures and was thus not considered for this project. In my research, I made use of *Amazon Web Services'* (AWS) FPGA development platform and workflow, which uses AMD's Xilinx platform to develop virtual FPGA's and deploy them to a cloud-based FPGA through AWS's F1 instances. For more information about using AWS for FPGA development, refer to the [AWS-FPGA github repo](https://github.com/aws/aws-fpga). 

As previously mentioned, we will be using AMD's Xilinx platform to develop and deploy FPGA code, however the actual Xilinx tool that we will be using is called "Vitis". While the AWS-FPGA github does describe other tools like "Vivado" or "SDAccel", each of these tools has a different workflow, as described in the [Development Environments](https://github.com/aws/aws-fpga#development-environments) section of the AWS-FPGA github. The main reason that we are using Vitis for this project is because Vivado and certain features of SDAccel (most importantly, OpenCL compatibility for acceleration) are built-into Vitis. Vitis also seems to be the most up to date set of tools provided by Xilinx for Software Developers interested in accelerating code on FPGA's. Vivado is similar to Vitis, but was created for developers using hardware-level langauges (Verilog, VDHL). For more information regarding Vitis, Vivado, or other Xilinx tools, refer to the official [Xilinx Documentation](https://docs.xilinx.com/home) site. The Xilinx documentation goes into much further detail about how [Vitis](https://docs.xilinx.com/r/en-US/ug1393-vitis-application-acceleration/Getting-Started-with-Vitis) works (which the AWS-FPGA github does not discuss). 

## Setting up AWS

While it is possible to complete all steps of this project with a secondary or sub AWS account, I reccomend using a primary or adminstrator account as to best avoid issues with permissions, as not all permissions are easily visible in the AWS dashboard.

The essential steps for setting up AWS for this project are:
 
1. **Adding a Keypair to AWS to access instances**
2. **Creating an AWS S3 Bucket**
3. **Creating 2 AWS instances (a t-series and an f1-series instance).**  
4. **Configuring the "FPGA Developer AMI" and login keypair on both instances**
5. **Logging into and Configuring the AWS CLI on the t-series instance.**
6. **Verifying that the F1 instance is also accsssible.** 

There are 2 main AWS instances required to get started. One instance can be any t-series or other general computing Virtual Machineinstance, while the second instance must be an f1-series instance, this "f1" instance is the FPGA based Virtual Machine (VM). 

*Note: We encountered an issue where we were unable to create both the T-series and the F1-series instances because of a vCPU limit set on all AWS accounts by default. [These](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-on-demand-instance-vcpu-increase/) AWS instructions should describe how to increase the limit and how to calculate what the new limit should be set to*.

**Step 1:** Before creating the instances, I reccomend adding an ssh key or "login keypair" to your AWS account as well as creating an S3 bucket. The keypair will make it easier to simply ssh into any AWS instances configured with that keypair. Generating SSH keys is platform dependent, but can typically be created using a terminal/command prompt, then copied into AWS. 

**Step 2:** The S3 bucket is required later on in the process to generate an Amazon FPGA image that can be deployed on the F1 instance. The instructions to create an S3 bucket can be found [here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html), I used the S3 console to create my bucket. Keep in mind that bucket names must be completlely unique to any other bucket on the AWS network from any other AWS user. 

**Step 3:** Once the access keypair and the S3 buckets are setup, we can start configuring our 2 instances. I reccomend the "t3.xlarge" or "t3.2xlarge" instance type for the general computing VM, the "t3.xlarge" instance I used took about 40 minutes per stage of FPGA compilation, so the "t3.2xlarge" instance should be much quicker. I also used a "f1.2xlarge" instance for the FPGA instance. The main reason for using a general-purpose instance and an F1 instance is to save cost. The FPGA code can be compiled into the FPGA image using the general-purpose instance then deployed with the host binaries and the executable on the "f1" instance.

**Step 4:** While creating both instances, be sure to select the "FPGA Developer AMI" under the "Application and OS Image" settings as this will ensure the VM is deployed with the appropriate Xilinx tools. The "FPGA Developer AMI" can be found in the Marketplace/Community tab, simply searching for "FPGA" should yield the correct result. If there are multiple results for "FPGA Developer AMI", select the image that uses CentOS, as it is what I used for this project. The last step is to add an existing keypair to the instance or create a new keypair and ensure that it is accessible on the local machine. 

If all steps are completed correctly, then new instances should be available in the AWS EC2 dashboard under "instances", or a new window stating that the new instance is running should open automatically. Once the instance starts running and shows the status "running", the machine can then be SSH-ed into. *Note: The FPGA Developer AMI does not work with the AWS Instance Web Launcher, it must be accessed another way.*

**Step 5a:** To access the machine, simply copy the instances public or external IP address and type the following command into a terminal/Powershell/Command Prompt: 

        ssh centos@[ip-address]

If you are prompted to accept SSH fingerprinting, simply type 'y' or 'yes', then you should have access to the instance (remote machine) from your terminal (local machine). 

**Step 5b:** Once you have created and have access to the t-series or general computing instance, the final step is to configure the AWS CLI. THis only needs to be done on the t-series or general-computing instance, not the f1 instance. [These](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) instructions will guide you on what you need to enter into your t-series instance to configure the AWS CLI. Once you have generated and stored your access ID and secret key, you must enter it into your instances by typing the following command into your terminal connected to the AWS instance:

        aws configure

After you enter the `aws configure` command, you should see the following prompt, simply enter the generated access ID and secret key, as well as your region (found in the top right corner of the AWS Dashboard):

        AWS Access Key ID [None]: [Enter generated access ID]
        AWS Secret Access Key [None]: [Enter generated secret key]
        Default region name [None]: [Enter region]
        Default output format [None]: json

As long as no errors appear immediately after completing this step, your instance should be ready to start creating and deploying FPGA images containg accelerated FPGA code. I reccomend logging into the f1 instance and ensuring that it can be accessed before continuing. 

## Compiling and Deploying FPGA code to an F1 Instance 	

The following are general instructions for compiling FPGA code into the requisite FPGA binary files (.xclbin) and executable into an Amazon FPGA Binary File (.awsxclbin). Both the executable and the Amazon FPGA Binary file are needed to deploy code on the f1 instance. 

1. Clone the AWS-FPGA git repo to the AWS instance's home directory. This should be done on both the t-series and the f1 instances *Optional: run `screen`*. 

        git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR

2. Navigate to the main directory inside the cloned AWS-FPGA git and enter the following command (Note: this must be run every time you restart the instance):  
 
        source vitis_setup.sh

3. Allow the Vitis setup script to run to completion and Run `screen`if you have not already. Ensure that only one TARGET is entered. Both `sw_emu` and `hw_emu` will run emulations of the binaries for debugging purposes, while the `hw` TARGET will build the Host application and FPGA binaries. To build or emulate the FPGA binaries run the following commands:
 
        cd $VITIS_DIR/directory/containing/fpga/source_files
        make clean
        make TARGET=[sw_emu, hw_emu, or hw] DEVICE=$AWS_PLATFORM all 

4. Upon successful emulation and/or compilation, the Host application (usually has no extension) and the FPGA binary files (.xclbin file) will be generated, keep track of these two files. 
5. The final step is to generate the Amazon FPGA Image, do so by running the following commands:  *(Note: AFI Generation executes in the background on a different machine within the AWS network and continues even if the instance used to generate the AFI request is offline, the AWS-CLI must be invoked to ascertain the status of the AFI generation.)*  
 
        cd $VITIS_DIR/tools
        ./create_vitis_afi.sh -xclbin=/path/to/.xclbinfile -o=NameForOutputFolder -s3_bucket=unique_s3_bucket_name -s3_dcp_key=s3dcpFolderName -s3_logs_key=s3LogsFolderName

6. Before moving the FPGA binaries to the F1 instance, you must first verify that the generation is actually complete, as the .awsxclbin file will be available on in the tools folder as soon as you run the generation commmand. To verify the status of the Amazon FPGA Image, you must first find the AFI ID of the specific .xclbin binary you are trying to convert. The AFI can be found by reading one of the text files ending in `_afi_id.txt` which can be found in the `$AWS_FPGA_REPO_DIR/Vitis/tools` directory. The line "FpgaImageId" line is the AFI ID. To check the status of an AFI using its AFI ID, run the following command (the "state" line should read "available" when the AFI has been fully generated): 

        cat [generated filename]_afi_id.txt
        aws ec2 describe-fpga-images --fpga-image-ids [AFI ID]

7. Transfer the compiled executable (my executable was found in the corresponding example folder) and .awsxclbin file from the tools directory to the local machine, using scp or an [external client](https://docs.aws.amazon.com/transfer/latest/userguide/transfer-file.html).
8. Start and access the F1 instance, then clone the github with the same command in step 1 (also seen below), then run the Vitis runtime setup script: 

        cd $AWS_FPGA_REPO_DIR [directory containing aws-fpga git repo]
        source vitis_runtime_setup.sh

9. To execute an f1 program, simply `./` the file_name of the executable and the ".awsxclbin" file. I reccomend creating a new folder on the home directory or another place to contain each program run. 

        ./[executable_name] ./[FPGA_Binary_File].awsxclbin

*Note: You may receive a `Permission Denied` error after running an executable on the f1 instance, simply run `chmod +x ./[executable_name]`*

## Useful Links and Sources
- AWS-FPGA Github Repo: [https://github.com/aws/aws-fpga](https://github.com/aws/aws-fpga) (Documentation for FPGA Development on AWS)
- AMD Xilinx Documentation: [https://docs.xilinx.com/home](https://docs.xilinx.com/home) (Xilinx-specific documentation, includes links for Vitis & Vivado)
- AMD Xilinx Wiki: [https://xilinx-wiki.atlassian.net/wiki/spaces/A/overview](https://xilinx-wiki.atlassian.net/wiki/spaces/A/overview) (General Xilinx FPGA and other embedded platform information)
- AMD Xilinx University Program Vitis Tutorial: [https://xilinx.github.io/xup_compute_acceleration/index.html](https://xilinx.github.io/xup_compute_acceleration/index.html) (Tutorials on using Xilinx's Vitis tools and other general information)
- AMD Xilinx Vitis Examples Page [https://xilinx.github.io/Vitis_Accel_Examples/2022.1/html/index.html#0](https://xilinx.github.io/Vitis_Accel_Examples/2022.1/html/index.html#) (List of Xilinx examples in the AWS-FPGA Repo and explanations for each example)
- HPC Challenge FPGA Applications Github: [https://github.com/pc2/HPCC_FPGA](https://github.com/pc2/HPCC_FPGA) (FPGA Benchmark suite using HPC challenge programs)

