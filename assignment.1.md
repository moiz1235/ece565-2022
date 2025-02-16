## ECE 565 Programming Assignment 1 Fall 2022
### Professor Mithuna Thottethodi

1. **Introduction**
    
    This assignment will serve as an introduction to the `gem5` simulator. It will walk you through the necessary steps to set up the simulator on your account, and provide you with some introductory tasks that will help you familiarize yourself with the `gem5` structure and source tree. While the tasks aren’t difficult in and of themselves, do not underestimate the time it will take to get familiar with the `gem5` source tree before you’ll be able to complete the tasks.
 The `gem5` simulator is written primarily in C++. However, configurations are done in Python, so you’ll need to be familiar with Python as well. It’s easy to read, but be aware that spacing in Python matters if you find yourself editing a Python file for the first time – it’s worth finding a quick tutorial on Python to learn the basics. A utility called SWIG is used to combine the configurations in Python and the actual simulator written in C++.
    
1. **Setup**
    The programming assignment will be completed on a cluster of "qstruct" servers. There are 19 of these machines. You can access them by ssh’ing into qstruct.ecn.purdue.edu from any terminal. You will use your Purdue Career Account username and password. Below is the sample command for logging in:
        
    ```console
    ssh <your-career-user-id>@qstruct.ecn.purdue.edu
    ```
    
    You will clone a repository that contains a customized version of the gem5 simulator into your own accounts and do all your programming in your local copies.
 
    ```console
    git clone /home/yara/mithuna2/gem5-Fall2022
    ``` 

    Do **not** clone the repository directly from gem5.org. The ECE-565 customized version has changes for compatibility with ECN-installed libraries. 
   
    (Git is the world's most popular revision control system, so if you are not familiar with it, now is a great time :).
    Given it's popularity, it is easy to find high quality documentation/tutorials online. This assignment document will provide you with the basic commands, but it is a good idea to become broadly familiar with git.)
    
    You now have your own fresh copy of gem5! Going into your gem5 directory, you’ll see a variety of folders, including the src directory, where most of your changes will be made. You may find yourself working in the configs directory from time to time as well. It is worth spending some time exploring these directories to get a feel for where different things are.
    
    <img src="gem5clone.png" width="50%" />
    
1. **Building gem5**
    
    The default gcc binaries and libraries available on ECN machines are not compatible with the version of gem5 we use. To use the non-standard version of gcc, some environment variables must be changed. 
    
    Modify your `PATH` environment variable to include: `/package/gcc/8.3.0/bin`
    Modify your `LD_LIBRARY_PATH` environment variable to include: `/package/gcc/8.3.0/lib64`
    
    (Note that the directories should be added to the beginning of the environment variables to ensure that the correct version of `gcc` is picked up by `scons-3` . The exact command to add these directories to the environment variable depends on the shell you use. It is your responsibility to figure out the appropriate steps.)

    gem5 is a highly configurable architectural simulator that supports a number of ISAs (x86, ARM, MIPS, SPARC, POWER, RISCV), CPU Models (InOrder, O3, AtomicSimple, TimingSimple, Minor (in-order)), and two Memory Models (Classic, Ruby). To understand how to build gem5, you must understand what you are building first. Example `gem5` build files are located in `gem5/build_opts`. For this assignment you will use the ECE565-specific build options from the ’ECE565-X86’ file whose content is as follows:
    
    ```console
    TARGET_ISA = ’x86’
    CPU_MODELS = ’AtomicSimpleCPU,TimingSimpleCPU,O3CPU,MinorCPU’
    PROTOCOL = ’MI_example’
    ```
    
    <img src="gem5buildopts.png" width="50%" />
    
    In this assignment, the MinorCPU is the relevant CPU model.
    
    We will be using the latest version of gem5, which has fairly up-to-date documentation outside of this assignment.
    For additional pointers on gem5, please see the book on learning gem5:
    
    https://www.gem5.org/documentation/learning_gem5/introduction/
    
    This file indicates that the target ISA is X86, and all of the CPU Models should be compiled in. The last line PROTOCOL is a specific type of coherence protocol for the Ruby Memory Model, which we will ignore for now.
For this assignment, we will use the ECE565-C86 and the ECE565-ARM build configurations. Now, the command to build this configuration is:
    
    ```console
    scons-3 USE_HDF5=0 -j `nproc` ./build/ECE565-X86/gem5.opt
    ```
    
    You may be missing the gem5 style hook. If so, just hit enter and let the script install the style hook for you. This build will take quite a while the first time around, so using multiple processes (using the ``-j `nproc` `` option) is valuable. The `nproc` argument results in the use of as many processes as number of processors in the system (40 on qstruct). Even so, it can take over 10 minutes to build. When you build the simulator with scons, all of your source code is copied into the gem5/build directory for that particular build. This means two things. First, you can always simply remove the build directory (or a specific build’s directory inside gem5/build) to start from scratch (think of it like a "make clean"). Second, never make any manual changes within the build directory. You should make your changes elsewhere (i.e. gem5/src), and re-build the simulator. Re-building gem5 after the first build typically only takes a minute or two, depending on your changes.

1. **Running gem5: Hello World**
    
    The gem5.opt file you built with scons is your binary – this is what you will use to run the simulator. It takes in options and a simulation script (this is where Python comes in). To get started, let’s run a Hello World program on the simulator. Looking in your gem5/configs directory, you’ll see directories including `example`. Take a look in the example directory. The se.py script will be what we need to run Hello World. Take a look at it. You won’t understand all of it just now, but you’ll see how it’s setting up different gem5 configurations.
    
    Now that we have a script to use with gem5, we need an actual Hello World binary to run on the simulator. gem5 comes with Hello World binaries already compiled for each ISA it supports. You can find them in gem5/tests/test-progs/hello/bin. Since we’ve built an x86 model, we’ll want the gem5/tests/test-progs/hello/bin/x86/linux/hello binary. 
    
    To actually run Hello World, go back to your gem5/ directory. The gem5 binary, the configuration script, and the hello binary are all used with the following command:
    
    ```console
    ./build/ECE565-X86/gem5.opt configs/example/se.py -c tests/test-progs/hello/bin/x86/linux/hello
    ```
   
    Notice that now there is a gem5/m5out directory. Inside, you’ll see a couple of configuration and stats files – they’re specific to your Hello World run. These will be very useful for tracking data in the future.
    
1. **Running gem5: Benchmarks**
    
    For the programming assignment, you’ll need to run benchmarks from the SPEC CPU benchmark suite. Runscripts for running select SPEC2006 CPU benchmarks are provided for you in `<gem5-root>/configs/spec/`. (If you don't see this directory, please pull from the git repo.) Specifically, the following six benchmarks have been tested on ECN machines: `sjeng`, `leslie3d`, `lbm`, `astar`, `milc`, and `namd`. The same runscript can be used to select and run any benchmark of your choice as follows:
    
    ```console
    ./build/ECE565-ARM/gem5.opt configs/spec/spec_se.py -b <benchmark-name>
    ```

    where `<benchmark-name>` must be replaced with any one of the six benchmark names listed above. You should use all six benchmarks for this homework. 
    

1. **Assignment**
    
    gem5 is an immensely complex piece of software with over 100k lines of code. However, it is not necessary to understand all of gem5 prior to being able to effectively use it. For this assignment you should focus on the *MinorCPU* Model located in /src/cpu/minor. The *MinorCPU* currently models a simple pipelined.  (Hint: Use the -h flag with your Python script to find out how to specify the CPU model you use.)
    One caveat of this CPU model is that each pipeline stage/component inherits from an abstracted resource object. As with every resource, there is the contingency of encountering a structural hazard. The occurrence of a structural hazard is dependent on the width of the corresponding stage. Thus in order to model one outstanding instruction per stage, you have to adjust the width of each stage to be 1 (*hint: look inside MinorCPU.py*).
    For this programming assignment, you will be required to implement the following changes in gem5:

    * Evaluate a simple pipeline.
    * Degrade branch prediction
    * Split the Execution stage into two separate pipeline stages
    
    Each of these are detailed further below. For part (i), just evaluate the example code, running to completion. For parts (ii) and (iii), you will need to run the SPEC benchmarks for 100 million instructions.  These results will then be compared to the baseline gem5 performance for the MinorCPU model. Make sure to take advantage of different output directories to avoid overwriting output data from different runs. 
    
    **NOTE:** Some tasks use the x86 ISA and others use the ARM ISA. The tasks/subtasks are organized such that the first few tasks use the x86 ISA and the remainder use the ARM ISA. This is annotated in the task section titles. You will have to rebuild gem5 to use the ARM target ISA for the later tasks.
    
    A nice overview of MinorCPU can be found here: https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu
    
    1. **Evaluate a Simple Pipeline**
    
        In this task - you will write a C program, compile it then run it using different CPU models. 
    
        1. **Run a custom program (x86)**
            
            The DAXPY loop (double precision aX + Y) is an oft used operation in programs that work with matrices and vectors. The following code implements DAXPY in C++11.
            Place the code inside a new directory *part1/daxpy.cc*.
    
            ```cpp
            #include <random>
            #include <iostream>
            
            int main()
            {
                const int N = 1000;
                double X[N];
                double Y[N];
                double alpha = 0.5;
                std::random_device rd; std::mt19937 gen(rd());
                std::uniform_real_distribution<> dis(1, 2);
                for (int i = 0; i < N; ++i)
                {
                X[i] = dis(gen);
                Y[i] = dis(gen);
                }
            
                // Start of daxpy loop
                for (int i = 0; i < N; ++i)
                {
                    Y[i] = alpha * X[i] + Y[i];
                }
                // End of daxpy loop
   
                double sum = 0;
                for (int i = 0; i < N; ++i)
                {
                    sum += Y[i];
                }
                std::cout << sum;
                return 0;·
            }
            ```
            
        
            Your first task is to compile this code statically and simulate it with gem5 using the timing simple cpu.
            To compile the code on qstruct, use the following compiler arguments (for c++11 and optimizations):
                    
            ```console
            /usr/bin/g++ -O2 -std=gnu++11 daxpy.cc
            ```
            Compile the program with -O2 flag to avoid running into unimplemented x87 instructions while simulating with gem5. Report the breakup of instructions for different op classes. For this, grep for op_class in the file stats.txt. Run the compiled file using
            
            ```console
            ./build/ECE565-X86/gem5.opt configs/example/se.py -c <output-from-daxpy-build>
            ```
    
        2. **Examine the assembly (x86)**
            
            Generate the assembly code for the daxpy program above by using the -S and -O2 options when compiling with GCC. As you can see from the assembly code, instructions that are not central to the actual task of the program (computing aX + Y) will also be simulated. This includes the instructions for generating the vectors X and Y, summing elements in Y and printing the sum. When I compiled the code with -S, I got about 350 lines of assembly code, with only about 10-15 lines for the actual daxpy loop.

            Usually while carrying out experiments for evaluating a design, one would like to look only at statistics for the portion of the code that is most important. To do so, typically programs are annotated so that the simulator, on reaching an annotated portion of the code, carries out functions like create a checkpoint, output and reset statistical variables.
            
            The mechanism for annotating the code is a library of operates called `m5ops`. You have to build it first. To do so, you need to follow these steps.
            
            1. Pull the updates from the main course repository from the main gem5 directory. (You have to commit any uncommitted changes before you can pull.)

             ```console
                   git pull
             ``` 
             
            2. You will edit the C++ code from the first part to output and reset stats just before the start of the DAXPY loop and just after it. For this, include the file ./include/gem5/m5ops.h in the program. To do so, you should use the line `#include<gem5/m5ops.h"` line in your source file. Use the function `m5_dump_reset_stats()` from this file in your program. This function outputs the statistical variables and then resets them. You can provide 0 as the value for the delay and the period arguments. This edited source file is now ready. But you cannot compile it yet until you have first built the m5ops library.
            3.  Build the m5ops library using the following command in the `<gem5-Fall2022>/util/m5` directory:
            
            ```console
                scons-3 build/x86/out/m5
            ```
            
            4. Now you can compile the DAXPY code (assuming the filename is daxpy.cc) using the following command from within the part1 directory. (If your source code is in another directory, please modify the include path and library path accordingly.)
                
            ```console
                /usr/bin/g++ -O2 -std=gnu++11 -I ../include -L ../util/m5/build/x86/out/ daxpy.cc -lm5
            ```

            <details>
                <summary> Old incompatible instructions for m5ops preserved here. Do not use.</summary>
    
            To provide the definition of the m5_dump_reset_stats(), go to the directory util/m5/src/x86/ and edit the SConsopts in the following way:

            ```console
            diff --git a/util/m5/src/x86/SConsopts b/util/m5/src/x86/SConsopts
            index 8763f29..7be70a3 100644
            --- a/util/m5/src/x86/SConsopts
            +++ b/util/m5/src/x86/SConsopts
            @@ -27,7 +27,6 @@ Import('*')
               
            env['VARIANT'] = 'x86'
            get_variant_opt('CROSS_COMPILE', '')
            -env.Append(CFLAGS='-DM5OP_ADDR=0xFFFF0000')
                
            env['CALL_TYPE']['inst'].impl('m5op.S')
            env['CALL_TYPE']['addr'].impl('m5op_addr.S', default=True)
            ```
            
            Execute the following command in the directory util/m5:
                
            ```console
            scons-3 src/x86 
            ```
            
            This will create an object file named util/m5/build/x86/x86/m5op.o. 
        </details>

        Now again simulate the program with the timing simple CPU. This time you should see three sets of statistics in the file stats.txt. Report the breakup of instructions among different op classes for the three parts of the program. In the assignment report, provide the fragment of the generated assembly code that starts with the call to m5_dump_reset_stats() and ends m5_dump_reset_stats(), and has the main daxpy loop in between.
            
        3. **Examine CPU types (ARM)**
        
            There are several different types of CPUs that gem5 supports: atomic, timing, out-of-order, inorder and kvm. Let's talk about the timing and the inorder cpus. The timing CPU (also known as SimpleTimingCPU) executes each arithmetic instruction in a single cycle, but requires multiple cycles for memory accesses. Also, it is not pipelined. So only a single instruction is being worked upon at any time. The inorder cpu (also known as Minor) executes instructions in a pipelined fashion. It has the following pipe stages: fetch1, fetch2, decode and execute.

            Take a look at the file MinorCPU.py. In the definition of MinorFU, the class for functional units, we define two quantities opLat and issueLat. From the comments provided in the file, understand how these two parameters are to be used. Also note the different functional units that are instantiated as defined in class MinorDefaultFUPool.

            Assume that the issueLat and the opLat of the FloatSimdFU can vary from 1 to 6 cycles and that they always sum to 7 cycles. For each decrease in the opLat, we need to pay with a unit increase in issueLat. Which design of the FloatSimd functional unit would you prefer? Provide statistical evidence obtained through simulations of the unannotated daxpy. For these experiments, please use the pre-compiled arm binary for daxpy that's provided at `/home/yara/mithuna2/gem5-Fall2022/daxpy-armv7-binary`. (For all parts of the homework that use the ARM ISA, we are unable to cross-compile on ECN's x86 machines. As such, you **must** use precompiled binaries.)
            
            If you wish - you can use the following new configuration file that provides an example of how to extend the MinorCPU with options for op/issue latency. Other students have reportedly used other approaches instead of inheritance. Some have reported that they successfully copied the files and modified the copies. As long as you can (1) implement a new kind of CPU, and (2) implement the opLat/issueLat varying in the new CPU, you will have met the requirements. You are not bound to a specific method.
            
            ```python
            # -*- coding: utf-8 -*-
            # Copyright (c) 2015 Mark D. Hill and David A. Wood
            # All rights reserved.
            #
            # Redistribution and use in source and binary forms, with or without
            # modification, are permitted provided that the following conditions are
            # met: redistributions of source code must retain the above copyright
            # notice, this list of conditions and the following disclaimer;
            # redistributions in binary form must reproduce the above copyright
            # notice, this list of conditions and the following disclaimer in the
            # documentation and/or other materials provided with the distribution;
            # neither the name of the copyright holders nor the names of its
            # contributors may be used to endorse or promote products derived from
            # this software without specific prior written permission.
            #
            # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
            # "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
            # LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
            # A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
            # OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
            # SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
            # LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
            # DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
            # THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
            # (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
            # OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
            #
            # Authors: Jason Power, updates by Tim Rogers
            
            """ CPU based on MinorCPU with options for a simple gem5 configuration script

            This file contains a CPU model based on MinorCPU that allows for a few options
            to be tweaked. 
            Specifically, issue latency, op latency, and the functional unit pool.

            See src/cpu/minor/MinorCPU.py for MinorCPU details.

            """

            from m5.objects import MinorCPU, MinorFUPool
            from m5.objects import MinorDefaultIntFU, MinorDefaultIntMulFU
            from m5.objects import MinorDefaultIntDivFU, MinorDefaultFloatSimdFU
            from m5.objects import MinorDefaultMemFU, MinorDefaultFloatSimdFU
            from m5.objects import MinorDefaultMiscFU
    
            class MyFloatSIMDFU(MinorDefaultFloatSimdFU):
    
            # From MinorDefaultFloatSimdFU
            # opLat = 6
    
            # From MinorFU
            # issueLat = 1

                def __init__(self, options=None):
                    super(MinorDefaultFloatSimdFU, self).__init__()

                    if options and options.fpu_operation_latency:
                        self.opLat = options.fpu_operation_latency

                    if  options and options.fpu_issue_latency:
                        self.issueLat = options.fpu_issue_latency


            class MyFUPool(MinorFUPool):
                def __init__(self, options=None):
                    super(MinorFUPool, self).__init__()
                    # Copied from src/mem/MinorCPU.py
                    self.funcUnits = [MinorDefaultIntFU(), MinorDefaultIntFU(),
                                  MinorDefaultIntMulFU(), MinorDefaultIntDivFU(),
                                  MinorDefaultMemFU(), MinorDefaultMiscFU(),
                                  # My FPU
                                  MyFloatSIMDFU(options)]


            class MyMinorCPU(MinorCPU):
                def __init__(self, options=None):
                    super(MinorCPU, self).__init__()
                    self.executeFuncUnits = MyFUPool(options)
    
            ```
            
            To simulate workloads with the MinorCPU, you may use the following command (assuming that the daxpy-armv7-binary has been placed in `<gem5root>/part1`: 
            
            ```console
            ./build/ECE565-ARM/gem5.opt configs/example/se.py --cpu-type=MinorCPU --maxinsts=250000 --l1d_size=64kB --l1i_size=16kB --caches --l2cache -c part1/daxpy-armv7-binary
            ```
            
            The command includes several options which are worth getting familiar with. E.g., options to change the CPU types, options to limit simulation to a given number of instructions, and options to include/configure caches in the simulation.
            
    1. **Degrade Branch Prediction (ARM)**
        
        For this part and the following part (split execution stage) you should use the ARM build of gem5. Make sure you are building the ARM version using the command line:
        
        ```console
        scons-3 USE_HDF5=0 -j `nproc` ./build/ECE565-ARM/gem5.opt
        ```
        
        The MinorCPU already implements a few branch predictor modules, including a tournament predictor and a simpler Branch Target Buffer (BTB). The pipeline timing enables you to figure out at the EX stage whether or not the branch prediction was correct. What you need to do is implement an option that will allow you to not only enable/disable the branch predictor, but degrade its accuracy to different levels as well.
        
        Note that the pipeline already handles the squashing of instructions fetched from the wrong path. On a misprediction, the pipeline will initiate calls to update the corresponding branch predictor’s entry with the correct target address.
        
        It would be functionally correct to squash the pending instructions if the predicted target is correct, but it would not be functionally correct to consider the branch correctly predicted when the predicted target does not match the actual branch target, and still consider it to be correct. For this part of the assignment, you need to implement a feature that degrades the branch predictor’s accuracy by treating some correctly predicted branches as incorrect. You will need to generate a random number between 0 and 1, and for a specified accuracy of X%, if the generated number is greater than X%, consider it a misprediction. Note that this "accuracy" is not the branch predictor’s accuracy, but the percentage of it’s original accuracy. That is, 100% would be equal to the default branch predictor’s accuracy, 50% would be half of the original predictor’s accuracy, and 0% would be an always incorrect predictor – it always predicts the wrong thing.
        
        For this, you will need to run the benchmark simulations for those three "accuracies:"
       
        * 100%
        * 50%
        * 0%
        
    1. **Split Execution Stage (ARM)**
    
        For this part, we want to be able to split up the Execution Unit into two stages. Modern pipelines employ deeper pipelines in order to increase the clock frequency. Instead of having a more complex stage that requires additional cycles, the corresponding stage is split into smaller stages, where each requires fewer cycles. However, as you know there is a trade-off for every design decision. What you need to do is split up the EX stage into EX1 + EX2 accordingly. The simplest way to do this is to create a "dummy" stage in front of the current execute that does nothing but pass the input tot EX stage along.

1. **Submission instructions**
    
    The main deliverable of this assignment that will be graded is a report (and not the code). The code will still have to be submitted as a single patch for plagiarism checking. But the code is not what gets graded.
    
    Submit a report (maximum three pages) with graphs and/or tables to present results via brightspace. Each graph should summarize the results of one of the experiments. (Suggested format: Plot bar-graphs with benchmarks on the X-axis and IPC on the Y-axis. For each benchmark, show two or more bars; one is the baseline and the rest correspond to your changes.) The report is NOT meant to be an exercise in writing. I do not expect any text beyond a 2-3 sentence summary of the key observations for each graph. However, this is a minimum and not a maximum. If you have text you want me to see (e.g., assumptions, simplifications, data-gathering difficulties etc.), feel free to write additional text in the report. 
