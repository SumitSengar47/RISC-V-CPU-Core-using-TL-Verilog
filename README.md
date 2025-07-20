
# RISC-V-CPU-Core-Using-TL-Verilog
This repository contains the complete work developed during the "Microprocessor for You in Thirty Hours (MYTH)" Workshop, organized by VLSI System Design (VSD) in collaboration with Redwood EDA. In just five days, we explored the fundamentals of the RISC-V Instruction Set Architecture (ISA) and implemented a fully functional RV32I-based pipelined RISC-V CPU core using Transaction-Level Verilog (TL-Verilog).

The project is developed on the [Makerchip IDE](https://www.makerchip.com/) ,a browser-based platform designed for hardware modeling and simulation using [TL-Verilog](https://tl-x.org/). Alongside the CPU design, this repository also includes simple test programs written in C, Assembly, and pseudocode to validate the core’s functionality.

This workshop offers a practical and accelerated path into computer architecture and digital design, ideal for students, enthusiasts, and budding VLSI engineers.

   ![riscv cpu](https://user-images.githubusercontent.com/84727176/138549829-0a1ef365-6fe0-4a3b-8472-10c10c33a75e.png)


## Table of Contents
* [Introduction to RISC-V ISA](#Introduction-to-RISC-V-ISA)
* [About GNU Compiler Tool Chain](#About-GNU-Compiler-Tool-Chain)
* [Introduction to ABI](#Introduction-to-ABI) 
* [Digital Logic with TL-Verilog and Makerchip](#Digital-Logic-with-TL-Verilog-and-Makerchip)
* [RISC-V Core Implementation](#RISC-V-Core-Implementation)
    * [Program Counter](#Program-Counter) 
    * [Fetch](#Fetch)
    * [Decode](#Decode)
    * [Execute](#Execute)
    * [RISC-V Pipelined Core](#RISC-V-Pipelined-Core)
* [Conclusion](#Conclusion)
* [Acknowledgements](#Acknowledgements)
* [References](#References)
## Introduction to RISC-V ISA

An **Instruction Set Architecture (ISA)** defines the boundary between software and hardware. It specifies the set of machine-level instructions that a processor can execute, as well as the register set, memory model, and execution behavior. The ISA enables hardware engineers to design processor cores that follow a common specification, while software engineers build compilers, operating systems, and applications that run on top of it.

**RISC-V** is a modern, open-source ISA that follows the **Reduced Instruction Set Computer (RISC)** principles. It is designed to be simple, clean, and modular — ideal for both academic learning and industrial-scale processor development.

![RISCV-ISA](https://github.com/RISCV-MYTH-WORKSHOP/RISC-V-CPU-Core-using-TL-Verilog/blob/master/Documentation/Snaps/ISA%20_base_and_extensions.JPG?raw=true)

The RISC-V ISA consists of:
- A **base integer instruction set**, which is mandatory for all implementations
- Multiple **optional extensions**, such as `M` for multiplication/division, `A` for atomic operations, `F/D` for floating-point, and `C` for compressed instructions

Each base integer ISA is defined by:
1. **XLEN** – the width of the general-purpose integer registers (commonly 32, 64, or 128 bits)
2. **Address space** – corresponding to the XLEN (e.g., 32-bit address space for RV32)
3. **Register file** – a total of 32 general-purpose integer registers

In this workshop, we focus on the **RV32I** variant — the 32-bit base integer instruction set — which includes fundamental instructions for arithmetic, logic, memory access, and control flow.

RISC-V’s openness and simplicity allow it to be used freely in academia and industry, making it a powerful tool for innovation, customization, and learning.

📖 For more technical details, refer to the official [RISC-V ISA Specification (Draft)](https://github.com/riscv/riscv-isa-manual).

## About GNU Compiler Tool Chain
The GNU Toolchain is a popular set of programming tools commonly used in Linux systems. The toolchain contains GNU Make, GCC, GNU Binutils, GNU Bison, GNU m4, GNU Debugger, and the GNU build system. Each of these tools help programmers make and compile their code to produce a program or library. 

Under the risc-v toolchain, 
  * To use the risc-v gcc compiler use the below command:

    `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o<object filename> <C filename>`

    More generic command with different options:

    `riscv64-unknown-elf-gcc <compiler option -O1 ; Ofast> <ABI specifier -lp64; -lp32; -ilp32> <architecture specifier -RV64 ; RV32> -o <object filename> <C      filename>`

    More details on compiler options can be obtained [here](https://www.sifive.com/blog/all-aboard-part-1-compiler-args)

  * To view assembly code use the below command,
    
    `riscv64-unknown-elf-objdump -d <object filename>`
    
  * To use SPIKE simualtor to run risc-v obj file use the below command,
  
    `spike pk <object filename>`
    
    To use SPIKE as debugger
    
    `spike -d pk <object Filename>` with degub command as `until pc 0 <pc of your choice>`

    To install complete risc-v toolchain locally on linux machine,
      1. [RISC-V GNU Toolchain](http://hdlexpress.com/RisKy1/How2/toolchain/toolchain.html)
      2. [RISC-V ISA SImulator - Spike](https://github.com/kunalg123/riscv_workshop_collaterals)
    
    Once done with installation add the PATH to .bashrc file for future use.

![sum](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/df54356f05723174cb419d542d6629690bec8590/Images/1_sum.png)

![assembly](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/df54356f05723174cb419d542d6629690bec8590/Images/2_assembly.png)
## Introduction to ABI 

System programming involves designing and writing computer programs that allow the computer hardware to interface with the programmer and the user, leading to the effective execution of application software on the computer system. In order to achieve systems programming there needs to be an interface which communicates between software and hardware which is where the APPLICATION BINARY INTERFACE comes into play.

Application Binary Interface is an interface that allows application programmers to access hardware resources. RISC-V specification has 32 registers whose width is defined by XLEN which can be 32/64 for RV32/RV64 respectively.The data can be loaded from memory to registers or directly sent, Application programmer can access each of these 32 registers through its ABI name seen below

![abi](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/7076c8b7106eb2f4cb24b1444657ef4d2e5b45ee/Images/Application%20Binary%20Interface%20(ABI).png)


## Digital Logic with TL-Verilog and Makerchip
 <u>**Makerchip**</u>

[Makerchip](https://makerchip.com/) is a powerful, browser-based IDE developed by **Redwood EDA** for building, simulating, and debugging digital hardware designs. It provides a seamless workflow to **write TL-Verilog code**, **simulate behavior**, and **visualize waveforms** — all in one place, with zero installation required.

Makerchip was the central tool used throughout the MYTH Workshop, enabling real-time experimentation and rapid design iterations for all hardware modules.

<u>**TL-Verilog**</u>
 

**TL-Verilog (Transaction-Level Verilog)** is a modern extension of SystemVerilog that simplifies digital design by introducing **higher levels of abstraction**, **cleaner syntax**, and **built-in pipelining constructs**. It significantly reduces the code complexity while retaining the full expressiveness needed for real-world hardware.

Compared to traditional RTL design in SystemVerilog, TL-Verilog makes it easier to:
- Structure pipelines
- Manage flow control
- Design scalable, modular logic
- Reuse components

It is particularly well-suited for educational and early-stage processor development, making complex digital concepts easier to grasp and implement.

 <u>**Digital Design Learning**</u>

 
During the first few days of the workshop, I explored fundamental digital circuits using TL-Verilog on Makerchip. I implemented and simulated:
- **Basic logic gates** (AND, OR, NOT, XOR)
- **Combinational logic blocks**
- **Sequential circuits** using flip-flops and registers

These exercises solidified my understanding of how digital logic operates at a low level and how it's modeled in hardware description languages.

By the end of **Day 3**, I applied my knowledge to build a **sequential calculator** in TL-Verilog. This calculator could:
- Perform basic arithmetic operations
- Maintain state across clock cycles
- Demonstrate control flow using sequential logic

The calculator project served as my first complete digital system and helped bridge the gap between abstract logic and real CPU components.

 ![sequential Calc](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/34955150fdecb74086b81a08d4893889c156b31a/Images/3_sequential%20calculator.png)
# 🚀 RISC-V Core Implementation

This project involved building a RISC-V CPU core step-by-step using **TL-Verilog** on the **Makerchip IDE**, starting from a basic 3-stage core (Fetch → Decode → Execute) and evolving it into a fully functional **5-stage pipelined processor** based on the RV32I ISA.

The design journey was progressive — each stage was integrated and validated incrementally, resulting in better understanding of how processors work internally.

 ![riscvBD](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/4_RISCV-BD.jpg)


## Program Counter (PC)

The **Program Counter (PC)** is responsible for tracking the address of the current instruction. In the RV32I architecture, each instruction is 32 bits (4 bytes), so the PC increments by 4 every cycle unless modified by a branch or jump instruction.

Initially, we implemented a simple incrementing PC. Later, it was enhanced to handle branching logic based on instruction decoding.


![RISCV_CPU_PC_Implmentation](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/5_Program_counter_imp.png)



## Instruction Fetch

The **Instruction Fetch** stage retrieves the instruction from memory using the current PC value. It is the first stage of the CPU pipeline and forms the base of the instruction cycle.

In this stage:
- Instructions are fetched sequentially from memory.
- The fetch logic was designed to support uninterrupted pipelined execution.
- PC updates and instruction memory access were managed.


![CPU_Instruction_cycle_diagram](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/5_1Instruction_fetch.png)


## Instruction Decode

The **Decode** stage interprets the fetched instruction. It determines:
- Instruction type (R, I, S, B, U, J)
- Register operands (rs1, rs2, rd)
- Immediate values
- Control signals required for execution

As part of the decode process, the register file was introduced to read the necessary operands. Control logic for branch decision-making also starts here.

 _Decoding RISC-V Instruction Types_

 ![riscvISA](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/6_1_decodelogic.png)

![riscVDecodeLogic](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/6_Instruction_decode.png)

 _Waveform: Branch Instruction Decoding (BLT Signal)_

![Instruction_Decode_Waveform](https://user-images.githubusercontent.com/14968674/92879155-69db3980-f42a-11ea-9457-c2254b092e05.png)


## Execute

The **Execute** stage carries out operations defined by the instruction using the **ALU** (Arithmetic Logic Unit). This includes:
- Arithmetic operations like ADD, SUB, ADDI
- Logical operations like AND, OR, XOR
- Comparison operations like SLT (Set Less Than)

This stage finalizes the result of computation or branch decisions. Additionally, Makerchip’s **VIZ** tool was used to visually analyze instruction execution, making debugging and learning much more intuitive.

_ADDI Instruction Computation in ALU_

![ADD_register_write](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/7_2_Viz.png)


##  RISC-V Pipelined Core

The final stage was converting the 3-stage processor into a **5-stage pipelined CPU** by leveraging **timing-abstract modeling** in TL-Verilog. The five stages are:

1. **Fetch**
2. **Decode**
3. **Execute**
4. **Memory**
5. **Writeback**

Pipelining enables overlapping of instruction execution, improving throughput without increasing clock speed. Thanks to TL-Verilog’s abstraction, we achieved this transformation with minimal refactoring and without introducing timing bugs.

The final pipelined core executes a test program that computes the **sum of 9 numbers**, showcasing complete functionality.

🔗 [Code for pipelined implementation](https://github.com/RISCV-MYTH-WORKSHOP/RISC-V-Core/blob/master/Day3_5/risc-v_solutions.tlv)

 _RISC-V 5-Stage Pipelined Core_

![RISCV_CPU_CORE](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/7_1final_output.jpg)

 <u>**RISC-V-CPU-Core**</u>

![RiscV-CPU-Final](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/d61ba88763c15110b9639affb86ceb22cf1f956b/Images/7_3Final_Output_for_RISC-V_Implemented_CPU_Core.JPG)

---
