---
layout: distill
title: winning the cambridge university 3rd-year project prize award in 80+ hours (part 1).
description: a blog detailing my thought process behind optimizing a RISC-V processor as a beginner.
giscus_comments: false
date: 2023-07-16

authors:
  - name: Zhen Yuen Chong
    affiliations:
      name: University of Cambridge

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: About
  - name: My backgrond
  - name: Tools setup
  - name: Preliminary work and exploration
  - name: Design strategy

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }

---
## About

Every undergraduate under the Cambridge Engineering Tripos IIA (3rd Year) must undertake two projects towards the end of the academic year. My choices were the RISC-V Processor (this) and Machine Learning, with a recommended time-frame of 80 hours per project spread across 4 weeks. Yes, I apologize for the mildly misleading title for those who thought this was a hackathon 😄.

This project involved improving an unoptimized RISC-V processor running on an iCE40 FPGA in a tiny wafer-scale 2.15x2.50 mm WLCSP package, using a completely open-source toolchain (Yosys, Project IceStorm, NextPNR, etc). The baseline design provided is the Sail RISC-V processor to be optimized. The Sail RISC-V processor uses the RV32I instruction set architecture (ISA) and features a 5-stage pipeline. The iCE40 UltraPlus Mobile Development Platform Board consists of four onboard iCE4UP5K FPGAs, labeled A to D. However, only FPGA D will be considered in this project. All modifications were done in Verilog, and the final processor design was synthesized using Yosys. The project culminated in a competition between teams to achieve designs on the Pareto frontier for some selected competition binaries.

[GitHub link](https://github.com/zhenyuen/sail-riscv)

## My background

With an introductory understanding of computer organization and zero practical experience in Verilog, FPGAs, and open-source toolchains in the embedded space, I knew this project was going to be challenging right from the start. Furthermore, the information provided by our instructors and handouts was minimal at best, so I was mostly on my own. Given the limited time frame, compromises had to be made when implementing my proposed modifications while following the traditional FPGA design process. Hence, this blog is not meant to be educational but rather to shed some light on my thought process. If something feels too simple or "hacky," you are most likely right 😄.

Here is a list of the main resources or materials that helped me through this project:
- [Computer Organization and Design RISC-V edition: The hardware software interface](https://www.amazon.co.uk/Computer-Organization-Design-RISC-V-Architecture/dp/0128122757)
- [The RISC-V Instruction Set Manual](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf)
- [Foundations of Embedded Systems](https://f-of-e.org)

In addition, I would recommend my readers to learn how to read schematic diagrams. Most users (like me) have only used beginner-friendly embedded platforms such as Arduino or Raspberry Pi that come equipped with easy-to-read documentations and 'pretty' pin layouts. Credits to my friend, Ye Heng (do check out his [blog](https://zen.infinus-electronics.net/projects/)) for teaching me how to read the Lattice iCE40 FPGA pin-out diagram. The pins can be used to connect an oscilloscope to the FPGA, and the measured pin voltages can be used as flags during debugging.


## Tools setup

In the initial stages of this project, the required tools that needed to be set up were Project IceStorm, Yosys, NextPNR, GTKWave, and the Diligent WaveForms Tool. The build compilation process mainly follows these steps:
1. Using Yosys to synthesize the underlying hardware for the processor described in the Verilog.
2. Using NextPNR for placing and routing the components involved.
3. Using iceprog from Project IceStorm to configure the FPGA with the generated bit-stream.

Both Yosys and NextPNR can provide useful metrics on the simulated design, such as the total path delay, clock frequency, and device utilization.

Instead of having to build the toolchains locally, a [Docker image](https://github.com/f-of-e/gb3-resources) was provided, greatly simplifying the workflow by being pre-equipped with the aforementioned tools. Additionally, the use of a Docker container can avoid dependency conflicts by ensuring the desired run-time environment is used.

The Docker container also comes equipped with [Sunflower](https://github.com/physical-computation/sunflower-embedded-system-emulator), which is a full-system emulator. It can take compiled binaries and emulate them instruction by instruction for multiple complete embedded systems networked over wired or wireless connections, or integrated into a single chip and communicating over shared memory. Although the processor design used by Sunflower has no connection to the RISC-V processor I would be working on, it provides useful commands, such as obtaining the fetch instruction count for some compiled binary with the specified target architecture. Note that certain commands, such as `dumpdistr`, were avoided as they cause Sunflower to crash due to an unidentified bug.


## Preliminary work and exploration

The simplest metrics to evaluate a processor's performance would be the program execution time and the average cycle per instruction (CPI).
$$
\begin{equation}
\textit{Execution Time} = \frac{N_{instructions}}{N_{programs}} \cdot \frac{N_{cycles}}{N_{instructions}} \cdot \textit{Clock Rate}\label{eq:1__1}  
\end{equation}
$$

There are numerous ways to calculate the average CPI. The RV32I base integer instruction set architecture (ISA) consists of 47 unique in- structions and 31 general-purpose registers (x1-x31). The number of clock cycles per instruction can be obtained by using the RDCYCLE instruction, which reads the lower 32 bits of the cycle control status (CSR) register, holding the clock cycle count executed by the processor core. Alternatively, a comprehensive test bench can be written in Verilog, involving multiple modules such as the ALU and memory unit, working together to complete a full instruction pass through the pipeline (IF, ID, EX, MEM, WB). By simulating this test bench, the corresponding number of clock cycles can be obtained. However, due to time constraints, writing such a test bench is complex and infeasible.

Instead, to approximate the average CPI, a simpler method was employed. The execution time of a program is measured, and then divided by the dynamic instruction count. It is important to note that this approach has potential pitfalls, as the program contains a variety of different instructions with a non-uniform distribution in terms of their frequency and counts, which can distort the average CPI. Additionally, using different compiler versions or run- time environments may result in a different instruction mix. For more sophisticated benchmarks, interested readers can source them from other sources such as [FPBench](http://fpbench.org). Armed with the power of Sunflower and the rough assumption that the compiled binary for Sunflower and the RISC-V processor would be similar, the average CPI of my design could be measured. One may question the validity of my assumptions. After writing a variety of benchmarks in C and inspecting the corresponding assembly files generated via `objdump`, the static instruction counts for both targets are relatively similar, as shown in the Table below.

$$
\begin{table}
\center{}%
\caption{Benchmarks for baseline processor and sunflower\label{tab:preliminary}}
\begin{center}
\begin{tabular}{|c|c|c|c|c|}
\hline 
\textbf{Program} & $bubblesort$ & $B1$  & $B2$ & $B3$ \tabularnewline
\hline 
\textbf{Fetched instr. count} & 22.0M  & 9.0M & 275.7M & 10.5M\tabularnewline
\hline 
\textbf{Ex. time (Sunflower)} & 4.456  & 1.838 & 73.830 & 2.359\tabularnewline
\hline 
\textbf{Ex. time (FPGA)} & 5.269  & 2.203 & 37.700 & 2.694\tabularnewline
\hline 
\textbf{Average CPI} & 1.440  & 1.469 & 0.820 (1.607) & 1.539\tabularnewline
\hline
\end{tabular} 
\par\end{center}
\end{table}
$$

An interesting observation is that the average CPI computed for benchmark B2 is 0.820, which is lower than 1. This should not be possible for the baseline design, which is a single-core pipeline processor without super-scalar execution. Further investigation reveals that the static instruction count generated by the compiler for the Sunflower emulator is 25751, nearly twice the count for the FPGA, which is 12806 (this can be measured using the `wc` command by passing the assembly file generated by `objdump`). This disparity arises due to the use of multiplication operations in B2, which the hardware lacks support for. Consequently, the compiler performs software-level multiplication, which may differ depending on the flags and platform used.

In the baseline design, the instruction module’s implementation was left to the Yosys synthesizer, utilizing only the available logic cells (LC). However, if the target program exceeds the capacity of the LCs, the program fails to execute. Currently, the limit is set to 1024 instruction words, as indicated by the `program.hex` file. Each word corresponds to 32 bits or 1 byte. Nevertheless, the FPGA is equipped with an additional 30 EBR (Embedded Block RAM) blocks, each having a size of 4 Kbit, resulting in a total of 120 Kbit of memory. From my understanding, only the data memory is currently stored in the EBR. Consequently, this design choice of storing the instruction memory in the LCs can lead to varying critical path delays, depending on the program loaded onto the FPGA. Larger programs tend to consume more LCs and have longer routing paths, resulting in increased delays along the critical path. As a consequence, this lowers the upper limit of the stable clock frequency. Additionally, slight variations in the measured critical path delays between experiments may arise.


## Design strategy

The processor’s performance can be enhanced by either increasing its clock frequency or decreasing the clock cycles per instruction (CPI), both of which lead to reduced execution times per program, as shown in (1). The design strategy for achieving these improvements involves the following stages: (i) Understanding and identifying limitations in the baseline design. (ii) Improving or adding features to reduce CPI. (iii) Increasing the processor’s clock frequency by reducing the critical path delay.

The baseline processor is designed with a 5-stage pipeline, which includes the fetch (IF), decode (ID), execute (EX), memory access (MEM), and register write-back (WB) stages. Each instruction requires a minimum of 5 clock cycles to complete, resulting in a throughput of 1 CPI, where one instruction is completed per clock cycle. However, control hazards, such as those arising from conditional branches, prevent the processor from achieving a perfect CPI of 1. When a conditional branch is encountered, the condition needs to be evaluated at the EX stage to determine whether the branch should be taken or not for the next instruction. This introduces bubbles in the pipeline, where no operation (NOP) can be performed until the EX stage of the conditional branch is completed. To mitigate the impact of control hazards and reduce pipeline stalls, branch predictors are employed. Branch predictors make predictions about the outcome of conditional branches based on past branch behaviour, allowing the proces- sor to continue executing subsequent instructions based on the prediction. If the prediction is correct, the pipeline can proceed without stalls. However, if the prediction is incorrect, the processor needs to flush the incorrectly speculated instructions from the pipeline, incurring a penalty of 3 clock cycles in a 5-stage pipeline. Improving the accuracy of the branch predictor is essential to achieve a lower CPI. Accurate branch predictions help minimize pipeline stalls and the associated performance penalty, leading to improved overall performance.

The upper stable clock frequency limit of the processor is determined by the total delay in its critical path. Certain instructions are inherently more complex and may require more time to complete compared to others in the pipeline. This complexity can be due to factors such as involving more hardware, which incurs additional latency and communication costs. When such instructions occur, the pipeline stage han- dling that specific instruction becomes the bottleneck, and the other stages of the pipeline must be stalled until the bottleneck is cleared. Consequently, the clock frequency is limited by the time required for the bottleneck stage to finish its operation. To identify the bottleneck stage and analyze the total path delay, timing tools like icetime can be utilized. These tools provide information on the path taken by signals and help identify the parts of the processor that contribute to the total path delay. By analyzing this information, it becomes possible to identify components that are causing delays and explore alternatives to mitigate their impact on the critical path. For the baseline design, the upper stable clock frequency limit is approximately 14.69 MHz.

One approach to improve performance and increase the clock frequency is by increasing the number of pipeline stages and using smaller stages that require less time to complete. Many modern processors employ more than 5 stages in their pipeline, allowing them to achieve higher clock frequencies while maintaining a throughput of approximately 1 CPI. In our design, several modifications have been made to enhance the performance and resource usage of the processor. These modifications include adjustments to the clock frequency, improvements to the arithmetic logical unit (ALU), enhance- ments to the branch predictor, and refinements in the design of the pipeline stages. Each modification has its degree of success in improving overall performance. The main ideas behind these modifications involve removing redundant operations, optimizing the usage of available resources, and leveraging the onboard digital signal processors (DSP) present on the FPGA. By carefully considering these factors and making appropriate design choices, it is possible to achieve a Pareto optimal design that balances performance and resource usage with minimal trade-offs between the two objectives.

To be continued ...
