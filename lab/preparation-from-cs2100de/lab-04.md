# Lab 04

## Introduction

### What is RISC-V?

As the name implies, RISC-V is a Reduced Instruction Set Computing (RISC) architecture. Today, the lines between RISC and CISC (Complex Instruction Set Architecture) are blurred, but the biggest differentiator is that CISC architectures typically have fewer general-purpose registers, and allow instructions to perform logical operations on values in memory.

### Registers

All the registers have a name that describes the function they should be used for. The functions are described in the [RISC-V Application Binary Interface (ABI) specification](https://lists.riscv.org/g/tech-psabi/attachment/61/0/riscv-abi.pdf). The ABI describes how an operating system, such as Linux, running on a RISC-V CPU should use the registers. The table showing what each register is used for can be found [here](https://wenbo-notes.gitbook.io/ddca-notes/textbook/architecture/assembly-language#the-register-set).

### RISC-V Assembly Language

Assembly language is not a single language. **Every ISA has its own assembly language**, which is just a human-readable, text-based version of the machine code that the CPU will run later. So, RISC-V has its own assembly language, and so does ARMv8-A, x86-64, IA-64, MIPS, Z80...

In assembly language, every line has a **one-to-one mapping** to a set of **one or many machine language instructions**. That is, one line of assembly may map to 3 lines of machine code, but it will always map to all three of those lines.

{% hint style="success" %}
For a reference for every instruction used in RISC-V, [this document](https://www.cl.cam.ac.uk/teaching/1617/ECAD+Arch/files/docs/RISCVGreenCardv8-20151013.pdf) from University of Cambridge is pretty useful!
{% endhint %}

#### Logical and Arithmetic Instructions

> This part is mainly covered in the [textbook](https://wenbo-notes.gitbook.io/ddca-notes/textbook/architecture/assembly-language). I will just include some exceptions.

Most arithmetic and logic instructions need three operands: a destination and two sources. However, there are two exceptions:

{% code lineNumbers="true" %}
```armasm
neg x10, x11 # x10 will be set to -x11
not x10, x11 # x10 will be set to the bitwise negation of x11
```
{% endcode %}

* `neg` is the arithmetic negation function and performs a two's complement negation.
* `not` is the binary negation function and flips every bit (i.e. performs a one's complement negation).

## Warm-up Activity

As the name suggested, this is just a warm-up and the RISC-V assembly code is as follows,

{% code title="loop.asm" lineNumbers="true" %}
```armasm
.text
	lw s0, N
	la s1, A
	addi s2, zero, 0 # i = 0
	addi s4, zero, 0 # mul = 0
	
	for:
	bge s2, s0, done # i >= N?
	lw s3, 0(s1) # t = A[i]
	addi s4, s4, 2 # mul += 2
	add s3, s3, s4 # t += mul
	sw s3, 0(s1)
	addi s1, s1, 4 # increment address by 4
	addi s2, s2, 1 # i++
	j for

	done:
	
.data
	N: .word 10
	A: .word 10 20 30 40 50 60 70 80 90 100
```
{% endcode %}

## Main Assignment

> Will do it once I have time 😂
