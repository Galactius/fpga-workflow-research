# FPGA Research Documentation

All documentation for this project was written by ***Daniel Perez***, who was conducting supervised research with ***Dr. Michael Robson*** at *Villanova University*. 

***Daniel Perez*** is an Undergraduate Computer Science Major in the Class of 2024 at *Villanova University*. 

## Table of Contents

1. Background
2. Overall Project Overview
3. Summary of Spring 2022
4. Summer 2022 Planning
5. Sources

## Background

This project began in Spring 2022 during my Sophomore year, and continued into the Summer 2022, where I hired as a full-time research assistant during for that summer. While I was not able to present my research at Villanova's 2022 Research Symposium, I made sure to document the progress I had made since the project's inception. 

I began writing the documentation for this project on May 7th, 2022, and intend to continue adding to it until the project is complete and a paper has been published. 

An FPGA or Field-Programmable Gate Array, is essentially a processor with integrated reprogrammable logic gates that make running one program very efficient, as when fully compiled, a program will be baked into the FPGA's logic. This means that while an FPGA can only run one specific application at a time, it is capable of being reprogrammed to run a different application. This is in direct contrast to an ASIC or Application-Specific Integrated  Circuit, which boasts better performance overall, but cannot be reprogrammed to run different applications, its hardware must be completely tailored to run one singular program. 

That being said, developing either an ASIC or an FPGA can be costly for non-business ventures. My research focueses on FPGA's specifically due to the much lower cost of development as well the fact that an FPGA is generic, and allows me to freely reprogram it as needed. Should ASIC's become attainable for the average consumer/user, this research may need to be updated with new findings. 

In my research, I made use of Amazon Web Services (AWS)'s FPGA development platform and workflow, which involves using AMD Xilinx software to develop virtual FPGA's and deploy them to a cloud-based FPGA through AWS's F1 instances. Further details on this process can be found in the [add link here] Summary of Spring 2022 section.

The purpose of my research is to develop a tool or workflow that allows an individuals to easily and/or rapidly develop, test, and deploy FPGA's primarily through the AWS's Cloud F1 instances. As discussed in the Summary of Spring 2022 section, getting set up to run F1 instances is not necessarily easy, while plenty of documentation has been provided, some conflicting information makes it difficult to know exactly what is going on at any given phase of the development process. My goal with this project is to document and host a tool or workflow that simplifies this process as to increase the awareness and usage of FPGA's as this field of HPC is fairly novel and not well-known. 

## Overall Project Overview

## Summary of Spring 2022

Spring 2022 was the official start semester of this project, although I was persuing this project without being officially "hired on" as a Research Assistant. Another significant note is that this is original research that was not started by my supervisor, although Dr. Robson did contribute significantly to the organization of the project and as an all-round resource towards fixing any issues that arose during my research. 

After selecting AWS F1 FPGA Instances as my preferred platform for development and investigating its associated costs for usage, I began by first reading through the AWS-FPGA Development Github, which had most of the prerequisite information needed to get started. My first goal for this semster was to get become more comfortable with using AWS and the CLI interface through MACOS and WINDOWS. I primarily used MACOS, although I did temporarily switch to WINDOWS (with WSL) at various points during my research. Using virtual machines through AWS means that my research is not platform-dependent.

After getting my account setup through my supervisor's parent account, I spun up one "free-tier" instance in order to become acquainted with AWS and the AWS CLI API. During this first phase I also started setting up SSH keys, my process for file transfer using Filezilla, and how to navigate the AWS dashboard. Once I was comfortable using the CLI and AWS in my terminal and could confirm that my SSH keys were setup properly to authenticate as needed, I created a new, more powerful virtual machine to actually start building FPGA binaries, etc. 

The workflow for developing FPGA's involves: 
1. Starting an AWS instance with the AWS FPGA Dev. Image fromt the AWS Marketplace (this does not need to be the special "F1 FPGA" instance, I selected a cheaper, general purpose VM to save costs slightly). 
2. Binding the correct SSH keys from the host (local) machine to the remote AWS instance.  
3.  Configuring the AWS CLI API through both the AWS EC2 dashboard and entering information the instance's terminal using "aws-cli ...", as well as setting up an S3 bucket. 
4. Cloning the AWS-FPGA git to the AWS instance's home directory *Optional: run screen*. 
5. Navigating to the main directory inside the cloned AWS-FPGA git and sourcing the Vitis_setup,sh script. 
6. Validating (HW-emulation is preferred for some reason) and Compiling a OpenCL/C++ application into the FPGA executables and binary files.  
7. Upon successful validation and compilation, sourcing the AFI-generation script in the Vitis/Tools directory. 
8. Transferring the compiled executable (my executable was found in the corresponding example folder) and .awsxclbin file from the tools directory. *Note: AFI Generation executes in the background on a different machine within the AWS network and continues even if the instance used to generate the AFI request is offline, the AWS-CLI must be invoked to ascertain the status of the AFI generation.* 

## Summer 2022 Planning

## Sources

