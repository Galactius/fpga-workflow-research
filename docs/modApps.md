# Modifying FPGA applications to run on AWS F1 instances

## Important Considerations

While AWS F1 instances are supposed to act like any physical FPGA device, unless the specific appliation being acclerated is suited to the F1 instance, some modifications must be made to the application to get it to run on an F1 instance. The easiest applications to run on F1 instances are those that already have an FPGA implementation of some sort. If an application already has preconfigured settings for running on a Xilinx card, then the application will be much easier to setup on F1. We have not started trying to convert non-FPGA or Intel FPGA applications to F1 instances, but I believe that it would be significantly more complex and time-consuming to do so. The next important consideration is related to the type of FPGA memory that the application uses. There are two main types of memory that I've encountered while working on this project, primarily DDR (Double Data Rate, an application of the same DDR3/4/5 RAM used in consumer computer hardware) and HBM (High Bandwidth Memory). AWS F1 instances currently only make use of DDR memory, NOT HBM memory. This means that if a Xilinx FPGA implementation of an application or code exists, to port it to an AWS F1 instance, the application must also be configured to use **DDR** memory. The criteria for general kernels, applications, or codes that should be accelerated using FPGAs is still a topic being researched further. 

## High-Performance Conguate Gradient (HPCG-FPGA) Overview

We selected the [HPCG-FPGA benchmark](https://github.com/Xilinx/HPCG_FPGA) as a primary candidate for testing during this project because of two main reasons. The first reason was that the Github repo we found this benchmark on was created by Xilinx, so we would not have to create custom configuration files to get the benchmark running on our Xilinx-based AWS F1 instance. The second main reason was that beacuse the code was open source, if we needed to make any changes to the source code, we could do so. 

That being said, we did not realize that the benchmark runs on HBM memory until we actually tried running the benchmark on F1 and read at the source code during debugging. In order to get this benchmark running correctly, a few steps were required before it would actually run on the F1 instance: 

1. Change the kernel link settings (found in /src-fpga/[precision]/[kernel].ini) to use DDR memory rather than HBM. 
2. Change the kernel Compute source files for the executable (found in /src/Compute[Kernel]\_FPGA.cpp) to look for the DDR memory banks described in the link files rather than the pre-configured HBM memory.
3. Convert the .xclbin binary files into .awsxclbin using the method described in step 5 of the ["Compiling and Deploying FPGA code to an F1 Instance"](/techInfo/#compiling-and-deploying-fpga-code-to-an-f1-instance) section of [Getting Started](/techInfo) page. 

This section of the documentation will cover the first 2 steps in more detail. 

#### HPCG Kernel Link Settings

While the Vitis compiler can typicallly infer the correct connections to make, in the case of this benchmark, we need to manually select the correct connections for the vitis compiler to make so that it uses the DDR banks correctly. In this section, I will describe the most basic single kernel link settings that I used on the first revision of the benchmark. The performance numbers may not be very representative of the true performance of the FPGA, but as long as the single kernel link setting work, it should be possible for a user to improve the link settings later on. Our github repo will have the most current version of the link settings [here](Link Pending).

Because there are 4 kernels used in this benchmark: dp.ini, spmv.ini, symgs.ini, and waxpby.ini must all be edited. That being said, the general format is very similar between each of the kernels. 

The configuration file dp.ini for a single kernel DDR configuration file looks as follows: 

    [connectivity] 
    slr=dp_1:SLR0
    
    sp=dp_1.in1:DDR[3]
    sp=dp_1.in2:DDR[3]
    nk=dp:1

The "slr", "sp", and "nk" keywords are all "tags" according to Vitis, and describe some form of connectivity within the FPGA. The "slr" tag is used to set the SLR to be used in the kernel. According to the [AWS-FPGA Alveo to F1 Migration Docs](https://github.com/aws/aws-fpga/blob/master/Vitis/docs/Alveo_to_AWS_F1_Migration.md), the AWS F1 platform has 3 SLRs: SLR0, SLR1, and SLR2. SLR0 contains DDR3 (bank3), SLR1 contains DDR0 and DDR2 (bank0, bank2), SLR2 contains DDR1 (bank1). Keep in mind that the SLR and the correct DDR tag must correspond to one another, or some issues (ranging in severity) may arise when running the kernel. 

The "sp" tags are used to assign kernels to a DDR bank. The sp tags in this link file are composed of 3 parts. The kernel, the in/output for that specific kernel, and the ddr bank being assigned to said kernel. The name of this kernel is "dp", "dp\_1" refers to the name and specific kernel number. The ".in1/.in2" refers to the input assigned to this kernel, which is also described in the source files used to execute this kernel. The final part of this sp tag is the ":DDR[3]" section, which is the DDR bank assigned to this specific kernel. The final "nk" keyword is used to describe how many kernels are being created, where "dp" refers to the name of the kernel being accelerated, and the ":1" refers to the number of "dp" kernels being created. 

Adding multiple kernels to this link file is not difficult, but does require the developer to coordinate a few different pieces of information, primarily the name of the kernel, the number and location of the DDR bank, and the name of the accelerated functions inside the source code. To add a second kernel, we need to add a few lines to our link settings. The first being an "slr" tag with a second kernel and the DDR bank to be assigned to it, for example, `slr=dp_2:SLR1`. This line allocates a 2nd "dp" kernel to SLR1, which means that kernel "dp\_2" should only be assigned to DDR banks DDR0 and/or DDR2, according to the [AWS F1 specification](https://github.com/aws/aws-fpga/blob/master/hdk/docs/AWS_Shell_Interface_Specification.md). Next, we have to specify the corresponding "sp" tags with our second "dp" kernel. The line would look like `sp=dp_2.in1:DDR[2]`, make sure to also add a line for the .in2 function. Finally, simply update the "nk" tag with the number of kernels. 

The resulting file for the dp.ini configuation file for two kernels in DDR would look as follows: 

    [connectivity]
    slr=dp_1:SLR0
    slr=dp_2:SLR1

    sp=dp_1.in1:DDR[3]
    sp=dp_1.in2:DDR[3]
    sp=dp_2.in1:DDR[2]
    sp=dp_2.in2:DDR[2]
    nk=dp:2     

For the purposes of this example I arbitrarily chose the SLR and the DDR banks, the configuration files uploaded to our Github repo will be the most efficient (or highest-performing) combination of SLR and DDR tags in my testing. The benchmark may run when the DDR banks to not correspond to its SLR in the shell specification, but the vitis\_analyzer reccomends to use either the same SLR or an adjacent SLR. 

#### HPCG Source Code Modifications

As mentioned in the overview, in order to run the HPCG-FPGA benchmark on the F1 instance, we need to modify the "Compute" source code files for each kernel based on the link settings we specified. Typically, we would only alter the link settings but because this benchmark expects HBM memory to be used, we need to modify the source files that create the benchmark's executable.

In the [src](https://github.com/Xilinx/HPCG_FPGA/tree/main/src) folder of the HPCG-FPGA Github repo, the five files we need to edit are ComputeDotProduct\_FPGA.cpp, ComputeSPMV\_FPGA.cpp, ComputeSYMGS\_FPGA.cpp, ComputeWAXPBY\_FPGA.cpp, and common.hpp.

The general formula is the same for the 4 compute files: 

1. Change NUM\_KERNEL
2. Remove unnecessary BANK\_ITEMS from bank[]
3. Change flags for each of the buffers. 

Steps 1 and 2 are simple, for step 1 simply change the NUM\_KERNELS macro. It can be found at around line 60 for most of the files, we need to change this because the rest of the program's logic relies on using NUM\_KERNEL as a bound for various loops, make sure that this matches the same number of kernels in the corresponding link file. For step 2, we need to change the number of BANK\_ITEMS in the bank array at around line 82. Instead of having 31 BANK\_NAME items, we should have at most 4 BANK\_ITEMS, although I only reccomend having all 4 BANK\_ITEMS if you are using all DDR banks in the kernel's link file. We have to alter this bank[] of BANK\_ITEMS because this is used to refer to the FPGAs on-board memory and allocate the OpenCL buffers. Be sure to also change the number of elements in the bank array. 

The bank[] array for a single kernel DDR configuration using DDR[3] looks as follows: 

    #define BANK_NAME(n) n | XCL_MEM_TOPOLOGY //This line is included to show the MEM pointer
    const int bank[1] = {
    		BANK_NAME(3)}; 

If we were to specify a second DDR bank in the link settings, we would have to add another BANK\_NAME object with the corresponding bank name. BANK\_ITEM(3) refers to bank3 within the FPGAs memory, the same bank3 found when running `platforminfo xilinx_aws-vu9p-f1_shell-v04261818_201920_3`. Once again remember to change the number of elements in the array declaration.

Step 3 is also a relatively simple change, as previously mentioned, we need to change the flags for each of the allocated OpenCL buffers. The buffers can be found at around line 180 for each of the Compute source files. The line that declares these vectors is `std::vector<cl_mem_ext_ptr_t> inBufExt1(NUM_KERNEL);`, where "inBufExt1" refers to the name of one of the buffers on the OpenCL host. In the SMPV Compute file, for example, there are 3 host pointers: "inBufExt1", "inBufExt2", "outspmvBufExt". We need to change the flags for each of these host pointers. In the for loop immediately following the "vector\<cl\_mem\_ext\_ptr\_t\>" declaration, we need to change the .flags line for the "inBufExt1", "inBufExt2", "outspmvBufExt" vectors. 

My file for the SPMV kernel looks as follows: 

    inBufExt1[i].obj = source_in1[i].data();
    inBufExt1[i].param = 0;
    inBufExt1[i].flags = bank[i];    //This line changed from original. 

    inBufExt2[i].obj = source_in2[i].data();
    inBufExt2[i].param = 0;	
    inBufExt2[i].flags = bank[i];    //This line changed from original. 

    outspmvBufExt[i].obj = source_hw_spmv_results[i].data();
    outspmvBufExt[i].param = 0;
    outspmvBufExt[i].flags = bank[i];    //This line changed from original. 


The only thing I changed were the .flags lines, and I changed the i in bank[i ...].  

**IMPORTANT NOTE:** The order of the BANK\_NAME objects in the bank[] matters! The BANK\_NAME at a certain index of bank[] should match the kernel number. So, referring back to the link file, the kernel dp\_1 that makes use of bank DDR[3], the kernel number should and the DDR bank number should correspond to the correct BANK\_NAME object. 

A second option is to explictly state what BANK\_NAME object in the bank array when changing the flags for the buffers. 

We will also have to modify one of the common source files to point to the location on the F1 instance where the .awsxclbin files can be found, otherwise the host application will not be able to locate the FPGA binaries.

The last change we need to make is in the common.hpp file. Specifically, we need to change the "PATH\_TO\_BINARY\_..." macros (there are lines 68-71 in [this](https://github.com/Xilinx/HPCG_FPGA/blob/main/src/common.hpp) file. The path entered in these macros must be the path to the .awsxclbin files on your F1 instance. The .awsxclbin files must be uploaded to the F1 instance, and I reccomend figuring out where to store these files on your F1 instance before completing this step, as you must rebuild the host application in order for the host application to detect the kernels. Also make sure that "[kernel].awsxclbin" is at the end of the path rather than "[kernel].xclbin. 

The last steps before running HPCG are to compile the kernels into the binary files, generate the host executable, then generating the Amazon FPGA Image (AFI). Those instructions are the same instructions found in step 5 of the ["Compiling and Deploying FPGA code to an F1 Instance"](/techInfo/#compiling-and-deploying-fpga-code-to-an-f1-instance) section of [Getting Started](/techInfo) page. Refer to those instructions before continuing on to the execution step. 

#### Executing HPCG

Before uploading the .awsxclbin files and the host application to the F1 instance, I reccomend creating an "hpcg" folder on the home directory of the F1 instance before continuing, **make sure that this location matches the path specified in the common source file modified in the previous step**. Next, copy the host application and binaries to that created folder.

Before executing the benchmark, we need to install the MPI dependencies. Simply run the command `sudo yum install mpich-3.2-autoload mpich-3.0-devel`, then log out and back into the instance.   

After everything has been setup, to run the benchmark, enter the following commands: 

    mpirun -n 1 [path to xhpcg] 64 64 64

Note that the path to the xhpcg (host application/executable) can be absolute (from the root /) or relative (from the home directory ~/). The other parameters for executing the benchmark are as follows: 

    mpirun -n [number_of_mpi_nodes] [path_to_hpcg_executable] [dim_x] [dim_y] [dim_z]
