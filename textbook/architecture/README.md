# Architecture

The previous chapters introduced digital design principles and building blocks. In this chapter, we jump up a few levels of abstraction to define the architecture of a computer. The _architecture_ is the programmer's view of a computer. It is defined by

* the instruction set (language) and
* operand locations (registers and memory)

Many different architectures exist, such as RISC-V (our focus), ARM, x86, MIPS, SPARC, and PowerPC.

The first step in understanding any computer architecture is to learn its language. The words in a computer's language are called _instructions_. The computer's vocabulary is called the _instruction set_. All programs running on a computer use the same instruction set. Computer instructions indicate both the operation to perform and the operands to use. The operands may come from memory, registers, or the instruction itself.

Computer hardware understands only 1's and 0's, so instructions are encoded as binary numbers in a format called _machine language_. Just as we use letters to encode human language, computers use binary numbers to encode machine language. The RISC-V architecture represents each instruction as a 32-bit word. Microprocessors are digital systems that read and execute machine language instructions. However, humans consider reading machine language to be tedious, so we prefer to represent the instructions in a symbolic format called _assembly language_.

{% hint style="info" %}
The instructions sets of different architectures are more like different dialects than different languages.
{% endhint %}

A computer architecture does not define the underlying hardware implementation. For example, Intel and AMD both sell various microprocessors belonging to the same x86 architecture. These microprocessors can all run the same programs, but they use different underlying hardware. Therefore, they offer trade-offs in performance, price and power. Some microprocessors are optimized for high-performance servers, wheras others are optimized for long battery life in gadgets or laptop computers. The specific arrangement of registers, memories, arithmetic/logical units (ALUs), and other building blocks to form a microprocessor is called the _microarchitecture_ and will be discussed in the next chapter.

In this chapter, we start out journey into understanding the RISC-V architecture by introducing assembly language instructions, operand locations, and common programming constructs, such as branches, loops, array manipulations, and function calls. We then describe how the assembly language translates into machine language and show how a program is loaded into memory and executed.

Throughout the chapter, we describe how the design of RISC-V architecture was motivated by four principles articulated by David Patterson and John Hennessy in their text _Computer Organization and Design_:

1. regularity supports simplicity
2. make the common case fast
3. smaller is faster
4. good design demands good compromises
