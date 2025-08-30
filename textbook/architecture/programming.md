# Programming

In this section, we begin by discussing program flow and instructions that support high-level languages like C. Then, we explore how to translate the high-level constructs into RISC-V assembly code.

## Program Flow

Like data, instructions are stored in memory. Each instruction is 32 bits (4 bytes) long, as we will discuss later, so consecutive instruction addresses increase by four. For example, in the code snippet below, the `addi` instruction is located in memory at address `0x538` and the next instruction, `lw`, is at address `0x53C`.

<figure><img src="../../.gitbook/assets/risc-v-memory-instruction-example.png" alt=""><figcaption></figcaption></figure>

### Program Counter

The _program counter —_ also called the PC — keeps track of the current instruction. The PC holds the memory address of the current instruction and increments by four after each instruction completes so that the processor can read or _fetch_ the next instruction from memory. For example, when `addi` instruction is executing, PC is `0x538`. After `addi` instruction completes, PC increments by four to `0x53C` and the processor fetches the `lw` instruction at that address.

## Logical, Shift, and Multiply Instructions

The RISC-V architecture defines a variety of logical and arithmetic instructions. We introduce these instructions briefly here because they are necessary to implement higher-level constructs.

### Logical Instructions

RISC-V _logical operations_ include `and`, `or` and `xor`. These each operate bitwise on two source registers and write the result to a destination register, as shown in Figure 6.2. Immediate versions of these logical operations, `andi`, `ori`, and `xori`, use one source register and a 12-bit sign-extended immediate.

<figure><img src="../../.gitbook/assets/risc-v-logical-operations.png" alt=""><figcaption></figcaption></figure>

* The `and` instruction is useful for _clearing_ or _masking_ bits. (e.g., forcing bits to 0). See more from [NUS CG2111A Notes](https://wenbo-notes.gitbook.io/cg2111a-notes/studio/studio-1-gpio-programming#clear-certain-bits).
* The `or` instruction is useful for _set_ bits in a register (e.g., for bit to be 1). See more from [NUS CG2111A Notes](https://wenbo-notes.gitbook.io/cg2111a-notes/studio/studio-1-gpio-programming#set-certain-bits).
* A logical NOT operation can be performed with `xori s8, s1, -1`. Remember that -1 (0xFFF) is sign-extended[^1] to 0xFFFFFFFF (all 1's). XOR with all 1's inverts all the bits, so `s8` will get the one's complement of `s1`.

### Shift Instructions

_Shift instructions_ shift the value in a register left or right, dropping bits off the end. RISC-V shift operations are

1. `sll`: shift left logical
2. `slr`: shift right logical
3. `sra`: shift right arithmetic

{% hint style="info" %}
The difference and example of the above three operations can be seen from the [previous notes](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/arithmetic-circuits#shifters-and-rotators).
{% endhint %}

These shifts specify the shift amount in the second source register. Immediate versions of each instruction are also available (`slli`, `srli`, and `srai`), where the amount to shift is specified by a 5-bit unsigned immediate.

Figure 6.3 shows the assembly code and resulting register values for `slli`, `srli`, and `srai` when shifting by an immediate value. `s5` is shifted by the immediate amount, and the result is placed in the destination register.

<figure><img src="../../.gitbook/assets/risc-v-shift-instrucitons-example.png" alt=""><figcaption></figcaption></figure>

As discussed in the [previous notes,](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/arithmetic-circuits#shifters-and-rotators)&#x20;

* shifting a value by N is equivalent to multiplying it by $$2^N$$
* shifting a value right by N is equivalent to dividing it by $$2^N$$
  * **arithmetic right shifts** divide two's complement numbers, while
  * **logical right shifts** divide unsigned numbers

### Multiply Instructions

As we have seen in [previous notes](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/arithmetic-circuits#multiplication), multiplying two N-bit numbers produces a 2N-bit product. The RISC-V architecture provides various _multiply instructions_ that result in 32- or 64-bit products. These instructions are not part of RVI321 but are included in teh RVM (RISC-V multiply/divide) extension.

* The _multiply instruction_ (`mul`) multiplies two 32-bit numbers and produces a 32-bit product. `mul s1, s2, s3` multiplies the values in `s2` and `s3` and places the **least significant** 32 bits of the product in `s1`; the **most significant** 32 bits of the product are discarded.
* Three versions of the "multiply high" operation exist: `mulh`, `mulhsu`, and `mulhu`. These instructions put the high 32 bits of the multiplication result in the destination register.
  * `mulh` (multiply high signed signed) treats both operands as signed.
  * `mulhsu` (multiply high signed unsigned) treats the first operand as signed and the second as unsigned.
  * `mulhu` (multiply high unsigned unsigned) treats both operands as unsigned.

Using a series of two instructions — one of the "multiply high" instructions followed by the `mul` instruction — will place the entire 64-bit result of the 32-bit multiplication in the two registers designated by the user. For example, the following code multiplies 32-bit signed numbers in `s3` and `s5` and places the 64-bit product in `t1` and `t2`. That is `{t1, t2} = s3 x s5`.

{% code lineNumbers="true" %}
```armasm
mulh t1, s3, s5;
mul  t2, s3, s5;
```
{% endcode %}

## Branching

_Branch instructions_ modify the flow of the program so that the processor can fetch instructions that are not in sequential order in memory. They modify the PC to skip over sections of code or to repeat previous code.

* _Conditional branch_ instructions perform a test and branch only if the test is TRUE.
* _Unconditional branch_ instructions, called _jumps_, always branch.

### Conditional Branches

The RISC-V instruction set has six conditional branch instructions, eahc of which take two source registers and a label indicating where to go.

1. `beq` (_branch if equal_) branches when the values in the two source registers are equal.
2. `bne` (_branch if not equal_) branches when they are unequal.
3. `blt` (_branch if less than_) branches when the value in the **first** source register is **less than** the value in the **second**.
4. `bge` (_branch if greater than or equal to_) branches when the **first** is greater than or equal to the **second**.
   1. `blt` and `bge` treat the operands as signed numbers, while
   2. `bltu` and `bgeu` treat the operands as unsigned (the fifth and sixth conditional branch instructions in RISC-V)

Code Example 6.12 illustrates the use of `beq`. When the program reaches the branch if equal instruction (`beq`), the value in `s0` is equal to the value in `s1`, so the branch is taken. Thus, the next instruction executed is the `add` instruction just after the label called `target`. The `addi` and `sub` instructions between the branch and the label are not executed.

{% code title="Example 6.12 Conditional Branching Using beq" lineNumbers="true" %}
```armasm
 addi s0, zero, 4   # s0 = 0 + 4 = 4
 addi s1, zero, 1   # s1 = 0 + 1 = 1
 slli s1, s1, 2     # s1 = 1 << 2 = 4
 beq s0, s1, target # s0 == s1, so branch is taken
 addi s1, s1, 1     # not executed
 sub s1, s1, s0     # not executed
target:
 add s1, s1, s0     # s1 = 4 + 4 = 8
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. Assembly code uses _labels_ to indicate instruction locations in the program. A label refers to the instruction just after the label. When the assembly code is translated into machine code, these labels correspond to instruction addresses (as will be discussed later).
2. RISC-V assembly labels are followed by a colon (`:`).
3. Most programmers indent their instructions but not the labels to help make labels stand out.
{% endhint %}

In Code Example 6.13, the branch is not taken because `s0` is equal to `s1`, and the code continues to execute directly after the `bne` (branch if not equal) instruction. **All** instructions in this code snippet are executed. (Including the instruction under `target`)

{% code title="Example 6.13 Conditional Branch Using bne" lineNumbers="true" %}
```armasm
 addi s0, zero, 4   # s0 = 0 + 4 = 4
 addi s1, zero, 1   # s1 = 0 + 1 = 1
 slli s1, s1, 2     # s1 = 1 << 2 = 4
 bne s0, s1, target # s0 != s1? No (4==4), branch not taken
 addi s1, s1, 1     # s1 = 4 + 1 = 5
 sub s1, s1, s0     # s1 = 5 - 4 = 1
target:
 add s1, s1, s0     # s1 = 1 + 4 = 5
```
{% endcode %}

### Unconditional Branches

A program can jump — that is, unconditionally branch — using one of three instructions:

1. _jump_ (`j`): jumps directly to the instruction at specified label. See Code Example 6.14
2. _jump and link_ (`jal`): will be discussed later in function calls.
3. _jump register_ (`jr`): will be discussed later in function calls.

{% code title="Example 6.14 Unconditional Branching Using j" lineNumbers="true" %}
```armasm
 j target          # jump to target
 srai s1, s1, 2    # not executed
 addi s1, s1, 1    # not executed
 sub s1, s1, s0    # not executed
target:
 add s1, s1, s0    # s1 = s1 + s0
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. After the `j` instruction executes, this program unconditionally continues executing the `add` instruction at the label `target`. All of the instructions between the jump and the label are skipped.
{% endhint %}

## Conditional Statements

This section shows how to translate the high-level constructs (`if/else`, and `switch/case`) into RISC-V assembly language.

### If Statements

Code Example 6.15 shows how to translate an if statement into RISC-V assembly code.

{% tabs %}
{% tab title="High-Level Code" %}
{% code title="Example 6.15 If Statement" lineNumbers="true" %}
```c
if (apples == oranges)
    f = g + h;
apples = oranges - h;
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code title="Example 6.15 If Statement" lineNumbers="true" %}
```armasm
# s0 = apples, s1 = oranges
# s2 = f, s3 = g, s4 = h
    bne s0, s1, L1   # skip if (apples != oranges)
    add s2, s3, s4   # f = g + h
L1: sub s0, s1, s4   # apples = oranges - h
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
#### Code Explanation

1. The assembly code for the if statement tests the opposite condition of the one in the high-level code.
{% endhint %}

### If/else Statements

Code Example 6.16 shows an example `if/else` statement.

{% tabs %}
{% tab title="High-Level Code" %}
{% code title="Example 6.16 If/else Statement" lineNumbers="true" %}
```c
if (apples == oranges)
    f = g + h;
else
    apples = oranges - h;
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code title="Example 6.16 If/else Statement" lineNumbers="true" %}
```armasm
# s0 = apples, s1 = oranges
# s2 = f, s3 = g, s4 = h
    bne s0, s1, L1   # skip if (apples != oranges)
    add s2, s3, s4   # f = g + h
    j L2
L1: sub s0, s1, s4   # apples = oranges - h
L2:
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
#### Code Explanation

1. The assembly code tests for (`apples ≠ oranges`). If that opposite condition is TRUE, `bne` skips the if block and executes the else block. Otherwise, the if block executes and finishes with a jump (`j`) past the else block.
{% endhint %}

### Switch/case Statements

A case statement is equivalent to a series of nested if/else statements. Code Example 6.17 shows two high-level code snippets with the same functionality: they calculate whether to dispense $20, $50, or $100 from an ATM depending on the button pressed.

{% tabs %}
{% tab title="High-Level Code" %}
{% code title="Example 6.17 Switch/case Statement" lineNumbers="true" %}
```c
switch (button) {
    case 1: 
        amt = 20; 
        break;
    case 2: 
        amt = 50; 
        break;
    case 3: 
        amt = 100; 
        break;
    default: 
        amt = 0;
}

// equivalent function using
// if/else statements
if (button == 1) {
    amt = 20;
} else if (button == 2) {
    amt = 50;
} else if (button == 3) {
    amt = 100;
} else {
    amt = 0;
}
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code title="Example 6.17 Switch/case Statement" lineNumbers="true" %}
```armasm
# s0 = button, s1 = amt
case1:
    addi t0, zero, 1    # t0 = 1
    bne s0, t0, case2   # button == 1?
    addi s1, zero, 20   # if yes, amt = 20
    j done              # break out of case
case2:
    addi t0, zero, 2    # t0 = 2
    bne s0, t0, case3   # button == 2?
    addi s1, zero, 50   # if yes, amt = 50
    j done              # break out of case
case3:
    addi t0, zero, 3    # t0 = 3
    bne s0, t0, default # button == 3?
    addi s1, zero, 100  # if yes, amt = 100
    j done              # break out of case
default:
    add s1, zero, zero  # amt = 0
done:
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
#### Code Explanation

1. The RISC-V assembly implementation is the nearly the same as the high-level code snippet.
{% endhint %}

## Getting Loopy

### While Loops

The while loop in Code Example 6.18 determines the value of `x` such that $$2^x=128$$. It executes seven times, until `pow=128`.

{% tabs %}
{% tab title="High-Level Code" %}
{% code title="Example 6.18 While Loop" lineNumbers="true" %}
```c
// Determines the power of x such that 2^x = 128
int pow = 1;
int x = 0;

while (pow != 128) {
    pow = pow * 2;
    x = x + 1;
}
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code title="Example 6.18 While Loop" lineNumbers="true" %}
```armasm
# s0 = pow, s1 = x
    addi s0, zero, 1      # pow = 1
    add s1, zero, zero    # x = 0
    addi t0, zero, 128    # t0 = 128
while:
    beq s0, t0, done      # pow = 128?
    slli s0, s0, 1        # pow = pow * 2
    addi s1, s1, 1        # x = x + 1
    j while               # repeat loop
done:
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
#### Code Explanation

1. Like [if/else statements](programming.md#if-else-statements), the assembly code for while loops tests the opposite condition of the one in the high-level code.
{% endhint %}

### Do-while Loops

Code Example 6.19 illustrates such a loop.

{% tabs %}
{% tab title="High-Level Code" %}
{% code title="Example 6.19 Do/While Loop" lineNumbers="true" %}
```c
// Determines the power of x such that 2^x = 128
int pow = 1;
int x = 0;

do {
    pow = pow * 2;
    x = x + 1;
} while (pow != 128);
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code title="Example 6.19 Do/While Loop" lineNumbers="true" %}
```armasm
# s0 = pow, s1 = x
    addi s0, zero, 1      # pow = 1
    add  s1, zero, zero   # x = 0
    addi t0, zero, 128    # t0 = 128
while:
    slli s0, s0, 1        # pow = pow * 2
    addi s1, s1, 1        # x = x + 1
    bne  s0, t0, while    # pow != 128?
done:
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
#### Code Explanation

1. Unlike the [previous example](programming.md#while-loops), the branch checks the same condition as in the high-level code.
{% endhint %}

### For Loops

Code Example 6.20 adds the numbers from 0 to 9.

{% tabs %}
{% tab title="High-Level Code" %}
{% code title="Example 6.20 For Loop" lineNumbers="true" %}
```c
// Add the numbers from 0 to 9
int sum = 0;
int i;

for (i = 0; i < 10; i = i + 1) {
    sum = sum + i;
}
```
{% endcode %}
{% endtab %}

{% tab title="RISC-V Assembly Code" %}
{% code title="Example 6.20 For Loop" lineNumbers="true" %}
```armasm
# s0 = i, s1 = sum
    addi s1, zero, 0      # sum = 0
    addi s0, zero, 0      # i = 0
    addi t0, zero, 10     # t0 = 10
for:
    bge  s0, t0, done     # i >= 10?
    add  s1, s1, s0       # sum = sum + i
    addi s0, s0, 1        # i = i + 1
    j    for              # repeat loop
done:
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
#### Code Explanation

1. The for loop in the high-level code checks the `<` condition to continue, so the assembly code checks the opposite condition, ≥, to exit the loop.
{% endhint %}

[^1]: Sign-extended logical immediates are somewhat unusual. Many other architectures, such as MIPS and ARM, zero-extended the immediate for logical operations.
