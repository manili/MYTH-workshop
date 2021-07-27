# MYTH-workshop
A 5-day workshop to implement a RISC-V based processor named MYTH from both SW and HW aspects.

# Table of Contents
- [Tools needed for the workshop](#tools-needed-for-the-workshop)
  - [Overview of GNU compiler toolchain](#overview-of-gnu-compiler-toolchain)
    - [Install toolchain from the source code](#install-toolchain-from-the-source-code)
  - [Overview of Spike simulator](#overview-of-spike-simulator)
    - [Install Spike from the source code](#install-spike-from-the-source-code)
  - [Overview of iVerilog simulator](#overview-of-iverilog-simulator)
    - [Install iVerilog from package manager](#install-iverilog-from-package-manager)
- [Introduction to RISC-V ISA](#introduction-to-risc-v-isa)
  - [Hands-on lab 1](#hands-on-lab-1)
    - [RISCV GCC compile](#riscv-gcc-compile)
    - [RISCV GCC disassemble](#riscv-gcc-disassemble)
    - [Spike simulation and debug](#spike-simulation-and-debug)
- [Introduction to integer number representation](#introduction-to-integer-number-representation)
  - [Signed and unsigned numbers](#signed-and-unsigned-numbers)
  - [Hands-on lab 2](#hands-on-lab-2)
- [Introduction to ABI](#introduction-to-abi)
  - [Hands-on lab 3](#hands-on-lab-3)
    - [New algorithm for sumation of 1 to N using ASM](#new-algorithm-for-sumation-of-1-to-n-using-asm)
    - [Run a C program on RISC-V using iVerilog](#run-a-c-program-on-risc-v-using-iverilog)

# Tools needed for the workshop

There are some tools needed to be installed on your local computer for hands-on labs. Those include:
1. The GNU compiler toolchain
2. The Spike simulator
3. The iVerilog simulator

## Overview of GNU compiler toolchain

The GNU Toolchain is a set of programming tools in Linux systems that programmers can use to make and compile their code to produce a program or library. So, how the machine code which is understandable by processer is explained below.

  * Preprocessor - Process source code before compilation. Macro definition, file inclusion or any other directive if present then are preprocessed. 
  * Compiler - Takes the input provided by preprocessor and converts to assembly code.
  * Assembler - Takes the input provided by compiler and converts to relocatable machine code.
  * Linker - Takes the input provided by Assembler and converts to Absolute machine code.

### Install toolchain from the source code

Here are the steps to install the toolchain from the source code:

  ```
  $ cd
  $ git clone https://github.com/riscv/riscv-gnu-toolchain
  $ cd riscv-gnu-toolchain
  $ sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
  $ ./configure --prefix=/opt/riscv --with-arch=rv32gc --with-abi=ilp32d --with-arch=rv64gc --with-abi=ilp64d --with-arch=rv64i --with-abi=lp64 --prefix=/opt/riscv --enable-multilib
  $ sudo make -jN #Where N is the number of your cores e.g. 4
  $ export RISCV=/opt/riscv/
  $ export PATH=$PATH:$RISCV/bin
  ```

## Overview of Spike simulator

Spike is the golden reference functional RISC-V ISA C++ sofware simulator. It provides full system emulation or proxied emulation with HTIF/FESVR. It serves as a starting point for running software on a RISC-V target. Here is a highlight of some of Spikes main features:

  1. Multiple ISAs: RV32IMAFDQCV extensions
  2. Multiple memory models: Weak Memory Ordering (WMO) and Total Store Ordering (TSO)
  3. Privileged Spec: Machine, Supervisor, User modes (v1.11)
  4. Debug Spec
  5. Single-step debugging with support for viewing memory/register contents
  6. Multiple CPU support
  7. JTAG support
  8. Highly extensible (add and test new instructions)

[Reference](https://chipyard.readthedocs.io/en/latest/Software/Spike.html)

### Install Spike from the source code

Here are the steps to install the Spike from the source code:

  ```
  $ cd
  $ git clone https://github.com/riscv/riscv-isa-sim
  $ cd riscv-isa-sim
  $ sudo apt-get install device-tree-compiler
  $ mkdir build
  $ cd build
  $ ../configure --prefix=$RISCV
  $ sudo make -jN #Where N is the number of your cores e.g. 4
  $ sudo make install
  ```

## Overview of iVerilog simulator

Icarus Verilog (iVerilog) is a Verilog simulation and synthesis tool. It operates as a compiler, compiling source code written in Verilog (IEEE-1364) into some target format. For batch simulation, the compiler can generate an intermediate form called vvp assembly. This intermediate form is executed by the `vvp` command. For synthesis, the compiler generates netlists in the desired format.

### Install iVerilog from package manager

Here is the step to install the iVerilog from the APT package manager:

  ```
  $ sudo apt install iverilog
  ```

# Introduction to RISC-V ISA

A RISC-V ISA is defined as a base integer ISA, which must be present in any implementation, plus optional extensions to the base ISA. Each base integer instruction set is characterized by
  1. Width of the integer registers (XLEN) 
  2. Corresponding size of the address space
  3. Number of integer registers (32 in RISC-V)

More details on RISC-V ISA can be obtained [here](https://github.com/riscv/riscv-isa-manual/releases/download/draft-20200727-8088ba4/riscv-spec.pdf).

## Hands-on lab 1

In this lab the following snippet of code will be used for compilation and debug. Here is the code snippet:

  ```
  #include <stdio.h>
  
  int main() {
    int i, sum = 0, n = 5;
    
    for(i = 1; i <= n; i++) {
        sum += i;
    }

    printf("Sum of numbers from 1 to %d is %d\n", n, sum);

    return 0;
  }
  ```

The code snippet will be stored in a file named `sum1ton.c`. This file will be used during this hands-on lab.

### RISCV GCC compile

To compile the source code following template command will be used:

  ```
  $ riscv64-unknown-elf-gcc -O[COMPILER OPTIONS] -mabi=[ABI SPECIFIER] -march=[TARGET ARCHITECTURE] -o [OUTOUT FILE] [INPUT FILES]
  ```
  
[Here](https://www.sifive.com/blog/all-aboard-part-1-compiler-args) is the reference for the compiler options. So in this case here are two sample commands to compile the code:
  
  ```
  $ riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
  $ riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
  ```

### RISCV GCC disassemble

After compiling the source code it's time for disassembling and see what happend inside. Here our main question about difference of `-O1` and `-Ofast` options will be answered. Following picture shows the result of compiling the code with `-O1`:

  ![O1](Images/Lab1/O1.png)

And here is the result of compilation with `-Ofast` option:

  ![Ofast](Images/Lab1/Ofast.png)

It is observed that using `-Ofast` option may reduce the number of instructions from 15 to 12 campared to `-O1`. [This](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html) is a compelete reference for the GCC compiler optimization options.
