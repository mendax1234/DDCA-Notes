# Lec 03 - RISC-V ISA and Microarchitecture

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

The register set has been introduced in [Harris & Harris](https://wenbo-notes.gitbook.io/ddca-notes/textbook/architecture/assembly-language#the-register-set). But below are some useful notes

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
