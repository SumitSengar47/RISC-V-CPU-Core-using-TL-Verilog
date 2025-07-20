
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

## Introduction to ABI 

System programming involves designing and writing computer programs that allow the computer hardware to interface with the programmer and the user, leading to the effective execution of application software on the computer system. In order to achieve systems programming there needs to be an interface which communicates between software and hardware which is where the APPLICATION BINARY INTERFACE comes into play.

Application Binary Interface is an interface that allows application programmers to access hardware resources. RISC-V specification has 32 registers whose width is defined by XLEN which can be 32/64 for RV32/RV64 respectively.The data can be loaded from memory to registers or directly sent, Application programmer can access each of these 32 registers through its ABI name seen below

![abi](https://github.com/SumitSengar47/RISC-V-CPU-Core-using-TL-Verilog/blob/7076c8b7106eb2f4cb24b1444657ef4d2e5b45ee/Images/Application%20Binary%20Interface%20(ABI).png)

