# Lec 03 - RISC-V ISA and Microarchitecture

> As ISA is just architecture (see [Harris & Harris](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one#abstraction)!), so this lecture will talk about the architecture and microarchitecture part of RISC-V!

## RISC-V ISA

As we have already seen in [Harris & Harris](https://wenbo-notes.gitbook.io/ddca-notes/textbook/architecture)'s introduction to architecture, we first recap some important points regarding architecture and microarchitecture.

* Architecture: Programmer's view of computer
  * Defined by instructions & operand locations
  * Assembly language: human-readable format of    &#x20;instructions
  * Machine language: computer-readable format    &#x20;(1’s and 0’s)
  * Assembly language -> Machine language    &#x20;conversion is done by the assembler
    * one to one correspondence      &#x20;(except for pseudo-instructions)
* Microarchitecture: how to implement an architecture in hardware (we will see later in this lec)

### RISC-V Features

In the lec, we have introduced some interesting features about RISC-V, and they are as follows,

* As a RISC architecture, the RISC-V ISA is a load–store architecture — only load/store variants can access memory
  * No mixing of memory access with data processing or branching
* Interesting design choices to simplify hardware implementation
  * Especially the encoding of immediates (We will see later)

{% hint style="danger" %}
Instruction length and word length are not necessarily the same!
{% endhint %}

<details>

<summary>Instruction Length vs. Word Length</summary>

**Word length** is width of registers/ALU and it is determined by the architecture variant you choose. For example,

* **RV32** → word length = 32 bits (registers and ALU are 32-bit wide).
* **RV64** → word length = 64 bits.
* **RV128** (rare, theoretical) → word length = 128 bits.

**Instruction Length** is the number of bits to encode an instruction and it is determined by the ISA encoding design and extensions included. For example,

* Base ISA (RV32I, RV64I, RV128I) → instructions are **always 32 bits long**.
* If you add the **Compressed (C) extension**, some instructions are **16 bits long**.
* There are also **48-bit and 64-bit instruction encodings** in the RISC-V spec (for special extensions).

</details>

### Registers

The register set has been introduced in [Harris & Harris](https://wenbo-notes.gitbook.io/ddca-notes/textbook/architecture/assembly-language#the-register-set). But the following image adds the information about whether the register should be saved by the caller or callee.

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-register-set-full.png" alt=""><figcaption></figcaption></figure>

And below are some useful notes

* In Assembly code, we can use ABI name, like `zero`, `ra`, etc. But in compiled code, ISA names, like `x0`, `x1`, etc, is used.
* PC is not a register readable/writable explicitly by any  &#x20;instruction, e.g., it is not a visible register
  * In RISC-V, PC just stores the address of the current instruction. (Not like ARM, PC actually stores the address of the current instruction address +4 or +8)
  * Writing PC is done **only by** branch/jump instructions.
* **No** instruction updates more than one visible register, and the register updated is **explicitly** specified in the `rd` field.
  * This ensures that the [register file](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/memory-arrays#register-files) only needs **one write port**.
* **No** instruction reads more than two registers.
  * This ensures that the register file needs only **2 read ports**. Below is the example of RISC-V's register file,

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-register-file.png" alt=""><figcaption></figcaption></figure>

* There is **no flag registers**. Flags are never stored for “future use.” Instead, comparisons and branches are **self-contained**. For example, `beq x1, x2, target;` will directly compare `x1` and `x2`, and branches to `target`.
  * If you _do_ need the comparison result as data, use an instruction like `slt` to store it in an **general-purpose register** `slt x3, x1, x2;` means `x3 1` if `x1<x2`, else `x3=0`.
  * Because branch instructions already use the ALU for comparison, RISC-V usually has a separate unit to compute branch target addresses (`PC + offset`).

### Instruction Formats

RISC-V instruction has 6 formats, and they are well summarized in the following table,

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-instruction-format.png" alt=""><figcaption><p>This table is from <a href="https://github.com/jameslzhu/riscv-card">James Zhu</a> at UCB</p></figcaption></figure>

{% stepper %}
{% step %}
**Opcode field occupies the least significant part of the instruction**

This gives us the benefit that,

* CPU can recognize instruction type by just reading the first byte (since RISC-V uses [little-endian](https://wenbo-notes.gitbook.io/cg2111a-notes/studio/studio-13-communication-protocol#big-and-little-endianness) and little-endian puts opcode at the lowest address).
* "3L mnemonic" for little-endian: In **l**ittle-endian formatting, **l**east significant byte goes to **l**owest memory address.
{% endstep %}

{% step %}
`rd`, `rs1`, `rs2` **always in the same place across formats**

This makes decoder hardware is simpler (fewer multiplexers, will see later in the microarchitecture).
{% endstep %}

{% step %}
**All immediates are MSB-extended**

So all immediates become sign-extended words. RISC-V immediates are encoded in **2’s complement form**. So when you sign-extend, you’re effectively just preserving the correct integer value. For example, in RV32I, 12-bit immediate,

* Immediate field = `1111_1000` (0xF8).
* As 12-bit 2’s complement → −8.
* Sign-extend to 32 bits → `0xFFFFFFF8`.
* ALU sees this and just adds normally, giving a result 8 less.

{% hint style="danger" %}
Only all **immediates** are MSB-extended. Later we will see, value stored in registers sometimes are zero-extended.
{% endhint %}

However, this may become a bit wired with instruction `sltu`. But, let loop at the following example,

```armasm
sltiu x3, x1, -8
```

Suppose the above is our "set if less than immediate unsigned" instruction, and RISC-V processor will treate `-8` as `0xFFFFFFF8`.&#x20;

* `x1=-2`, `x1` will be interpreted as `0xFFFFFFFE`. As ALU just compares unsigned, `0xFFFFFFFE` is greater than `0xFFFFFFF8`, thus `false`, meaning under **unsigned comparison** `x1` is not smaller than `-8`!
* `x1=5`, `x1` will be interpreted as `0x00000101`. Similarly, ALU just compares unsigned, `0x00000101` is smaller than `0xFFFFFFF8`, thus `true`, meaning under **unsigned comparison** `x1` is **smaller** than `-8` (unsigned)

{% hint style="danger" %}
Treat negative number in 2's complement as **unsigned** meaning it will become a **very large positive number**.
{% endhint %}
{% endstep %}

{% step %}
**The location of immediate is well-designed**

From the table above, we may think why the immediate's location is that wired! Actually, later from the microarchitecture view, we will see that this is a **genius design** as it minimizes the number of multiplexers!
{% endstep %}
{% endstepper %}

### DP Instruction

"DP" stands for data-processing. The following tables summarises all the base DP instructions.

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-dp-instructions.png" alt=""><figcaption></figcaption></figure>

* `subi` is unnecessary as the assembler can encode `A-B` as `A+(-B)`. This find as `B` is immediate, if `B` is a register, then `B` cannot be known at assembly time, and that's why `sub` is still needed.
* Include the example of 3 types of shifts operation from Harris & Harris here!

<figure><img src="../.gitbook/assets/risc-v-shift-instrucitons-example.png" alt=""><figcaption></figcaption></figure>

#### DP Pseudo-Instruction

All the pseudo-instruction in RISC-V is introduced [here](../lab/resources/risc-v-resources.md#risc-v-pseudo-instruction)! Among them, knowing the working principle of `auipc` from [Lab 1](https://wenbo-notes.gitbook.io/ddca-notes/lab/lab-01-get-prepared#whats-actually-going-on-in-line-47-auipc) is also necessary!

#### Multiply and Divide

The multiply and divide are not part of the base instruction set, but are available as an optional standard extension.

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-multiply-divide.png" alt=""><figcaption></figcaption></figure>

### Memory Instruction

The following is the base memory instruction:

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-memory-instruction.png" alt=""><figcaption></figcaption></figure>

* For the memory instruction syntax, if `imm=0`, can omit the `imm`. e.g., `op rd, (rs1)`.
* For `lb` and `lh`, which loads a byte or a half word, the rest of bits are formed by **sign-extension/MSB-extension** (just copy the MSB) of the byte/half-word
* For `lbu` and `lhu`, **zero extension** (copy zero only) is done.

### Control Instructions

The following is the base control instructions:

<figure><img src="../.gitbook/assets/cg3207-lec03-risc-v-control-instruction.png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
**Jump vs. Branch**

1. `jal` can jump farther than conditional branches because
   1. `jal` instructions use 20-bit signed immediate
   2. Branch instructions use 12-bit signed immediate
2. `jal` allows for saving **return address**, while conditional branches cannot.
{% endstep %}

{% step %}
**Two types of jumps**

1. `jal`: jump and link, is a J-type instruction. And it stores return address in `rd`.
   1. Used when you know the **target address at assembly time**.
   2. Used in **function call** (Jump to a function’s code so it can execute)
   3. `imm` is 20-bit.
2. `jalr`: jump and link register, is an I-type instruction. And it stores return address in `rd` (usually `ra`)
   1. Use when the **target address is dynamic**, stored in a register.
   2. Used in **function return** (Go back to where the function was called from)
   3. `imm` is 12-bit, but it can jump anywhere in a 32-bit absolute address range. A LUI      \
      instruction can first load rs1 with the upper 20 bits of a target address, then JALR      \
      can add in the lower bits.
      1. Similarly, AUIPC then JALR can jump anywhere in a 32-bit         \
         pc-relative address range
{% endstep %}
{% endstepper %}

### Put it all together

Nothing is better than an example! So, let's loook at an example to put everything together.

{% tabs %}
{% tab title="High-Level C Code" %}
{% code lineNumbers="true" %}
```c
// Global variables
int z1;
int z2;

// Callee function: Computes and returns the sum of two integers
int fn2(int p, int q) {
    int temp = p + q;
    return temp;
}

// Caller function: Calls fn2 and performs additional operations
void fn1() {
    int x = 20;
    int y = 30;
    z1 = fn2(x, y); // Store result of fn2(x, y) in z1
    x++;
    y++;
    z2 = x + y; // Store x + y in z2
}
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code lineNumbers="true" %}
```armasm
# Caller function: fn1
# Calls fn2, increments x and y, and stores results in z1 and z2
fn1:
    # Prologue: Save callee-saved registers
    addi sp, sp, -4        # Allocate stack space
    sw s0, 0(sp)           # Save s0 to stack

    # Initialize variables
    li s0, 20              # x = 20
    li t0, 30              # y = 30

    # Save caller-saved registers
    addi sp, sp, -8        # Allocate stack space for t0 and ra
    sw t0, 0(sp)           # Save t0 (y) to stack
    sw ra, 4(sp)           # Save return address

    # Call fn2(20, 30)
    mv a0, s0              # Set first argument: p = x
    mv a1, t0              # Set second argument: q = y
    jal ra, fn2            # Call fn2, result in a0

    # Store result in z1
    sw a0, z1, t0          # z1 = fn2(x, y)

    # Restore caller-saved registers
    lw ra, 4(sp)           # Restore return address
    lw t1, 0(sp)           # Restore y to t1
    addi sp, sp, 8         # Deallocate stack space

    # Increment x and y
    addi s0, s0, 1         # x++ (x = 21)
    addi t1, t1, 1         # y++ (y = 31)

    # Compute and store x + y in z2
    add t2, s0, t1         # t2 = x + y
    sw t2, z2, t3          # z2 = x + y

    # Epilogue: Restore callee-saved registers
    lw s0, 0(sp)           # Restore s0
    addi sp, sp, 4         # Deallocate stack space
    ret                    # Return to caller

# Callee function: fn2
# Arguments: p in a0, q in a1
# Returns: temp (p + q) in a0
fn2:
    # Prologue: Save callee-saved registers
    addi sp, sp, -4        # Allocate stack space
    sw s0, 0(sp)           # Save s0 to stack

    # Compute temp = p + q
    add s0, a0, a1         # temp = p + q
    mv a0, s0              # Move result to a0 for return

    # Epilogue: Restore callee-saved registers
    lw s0, 0(sp)           # Restore s0 from stack
    addi sp, sp, 4         # Deallocate stack space
    jr ra                  # Return to caller
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. From this example, we see some useful pseudo-instructions, like `jr` (for function return) and `mv` (for register copy)
{% endhint %}
{% endtab %}
{% endtabs %}

#### Some useful resources for RISC-V ISA

1.
