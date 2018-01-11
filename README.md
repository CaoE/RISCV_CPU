# RISCV-CPU

This is a project of Computer System class of ACM honor class in SJTU.

Copyright (c) 2017 Zhanghao Wu

## Abstract

This project is a simple five stage pipelined cpu for risc-v (rv32i) written in verilog HDL. 

It has features as follows:

1. It can behave correctly for the instructions in rv32i, except ECALL, EBREAK, CSRRW, CSRRS, CSRRC, CSRRWI, CSRRSI, CSRRCI

1. It has a cache on board and a uart module to link the cpu to the memory simulator on PC.

1. It has exciting light and status indicator. RGB!!!

## Introduction

The data and control flow are in the graph below.

### Five stge pipline

The pipeline is similar to the standard mips five stage pipeline with forwardings. For more information, you can read the [Reference [1]](#ref1).

### Forwarding

Just as the graph above shows, each stage after ID has a wire connecting to it, to forward the data back. And in this design, ID stage can get the data in the same cpu cycle.

### Jumps and Branches

To get a better trade off of jumps and the clock rate, I tried a lot methods. And my work can be divided into 2 stages, no-memory-delay and great-memory-delay.

1. In my disgn, there are some wires from ID and EX to pc_reg and mid stage -- IF_ID and ID_EX, in order to set new address and clear the former instruction ( presice interrupt ).
	- no-memory-delay: PC_reg and mid stage update their status as soon as the signal reach, which works well when the IF stage never stall.
	- great-memory-delay: In this case the pc_reg and mid stages are disgned as state machine, when they get a signal for jump, they get into another state to save the signal.

2. I add switches for *ID_BRANCHES* and *ID_JALR*, indicating that the in which state the branches and jalr should be executed.
	- Firstly, the JAL can be safely executed at ID stage. And the other branches and jumps can be set at ID stage, when the IF stage will never stall.
	- However, as the branch instruction needs data from regeisters which may be forwarded from the stage after ID. More comparation and addition will make ID take more time, and that can be a bottle neck that make the CPU clock rate much slower.
	- Another reason I put the all of the execution of branches, execpt jal, to EX stage is that, when the IF and ME should stall for some cycles, and the pc_reg and mid stage should save the signal, the time for ID to get the forwarding data will be much longer, ID may give wrong branch signal before the needed data reach.
	- Further more, when the branches are set in EX, we can use branch prediction in ID, to speed up our pipeline (uncompleted).

	PS: Branches in EX, without branch prediction can be regarded as always predict not jump, when the instruction is recogonized by ID.

### IF and ME

The logic to wait data to be ready can be quite hard to implement in hardware, when sequential logic is needed. My first design will generate latch, which is strongly not recommanded. ( [Reference [3]](#ref3) )

As the cache will give a done signal when the data is ready and the signal only last for one cycle, the logical to wait for the cache to be done can be quite simple and amazing. ( In IF.v and ME.v you can see the code ).

### Cache and memory control

The cache and memory control of my design is from Zhekai Zhang's design ( [Referece [2]](#ref2) ). And the cache is modified by me to make it able to uncache the address 0x100


### Makefile for test generation

I write a makefile for make, which can make both .s and .cpp/.c files by using the riscv-tool-chain. The guidance for using it is in [Makefile for risc v tool chain](https://gist.github.com/Michaelvll/46e069e29a8448326acadd7bb2bb1654).

1. The imm in sltiu command is signed extended, and unsigned compared.

1. The aluop is firstly numbered by the opcode+funct3+(funct7?), then it is renumbered by the sequence: 0x1,0x2,..., to make the number shorter for better performance.

## Support ISA

Now this project support almost all of the commands in rv32i. The suppoted commands are listed in the [Appendix A](#ApdxA)

## Q&A

1. Why op_imm, like xori command doesn't support imm larger than 0x7ff such as 0x801?
- Because the imm is signed.

## Reference

1. <span id="ref1">[A MIPS CPU written in Verilog by jmahler](https://github.com/jmahler/mips-cpu.git)</span>
1. <span id="ref2">[A Mips CPU written in verilog by sxtyzhangzk](https://github.com/sxtyzhangzk/mips-cpu.git)</span>
1. <span id="ref3">[Always@](www-inst.eecs.berkeley.edu/~cs150/sp13/resources/Always.pdf)</span>
1. 《自己动手写CPU》雷思磊

## Appendix

### <span id="ApdxA">A. Suppoted commands</span>

| Command | Support |
|---------|---------|
| LUI     | [O]     |
| AUIPC   | [O]     |
| JAL     | [O]     |
| JALR    | [O]     |
| BEQ     | [O]     |
| BNE     | [O]     |
| BLT     | [O]     |
| BGE     | [O]     |
| BLTU    | [O]     |
| BGEU    | [O]     |
| LB      | [O]     |
| LH      | [O]     |
| LW      | [O]     |
| LBU     | [O]     |
| LHU     | [O]     |
| SB      | [O]     |
| SH      | [O]     |
| SW      | [O]     |
| ADDI    | [O]     |
| SLTI    | [O]     |
| SLTIU   | [O]     |
| XORI    | [O]     |
| ORI     | [O]     |
| ANDI    | [O]     |
| SLLI    | [O]     |
| SRLI    | [O]     |
| SRAI    | [O]     |
| ADD     | [O]     |
| SUB     | [O]     |
| SLL     | [O]     |
| SLT     | [O]     |
| SLTU    | [O]     |
| XOR     | [O]     |
| SRL     | [O]     |
| SRA     | [O]     |
| OR      | [O]     |
| AND     | [O]     |
| Fence   | [X]     |
| Fence.I | [X]     |
| ECALL   | [X]     |
| EBREAK  | [X]     |
| CSRRW   | [X]     |
| CSRRS   | [X]     |
| CSRRC   | [X]     |
| CSRRWI  | [X]     |
| CSRRSI  | [X]     |
| CSRRCI  | [X]     |