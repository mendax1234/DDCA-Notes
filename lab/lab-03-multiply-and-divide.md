# Lab 03 - Multiply and Divide

## Introduction

In this lab, we are still using the [single cycle RISC-V processor](preparation-from-cs2100de/lab-02.md) which we have designed in Lab 02. The only difference is that, we will incorporate the **multiply** and **divide** into our processor. So, to set up for this lab, you can just copy all the design files from your lab 02 and then copy the `MCycle.v` into your design files and `text_MCycle.v` into your testbench from [here](https://github.com/NUS-CG3207/labs/blob/main/docs/asst_manuals/Asst_03/lab3_Verilog_design.zip)!

## Task 1: Implement Division

> This task only needs **HDL simulation**.

For this task, what we do is to **strictly** follow the [sequential divider](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/arithmetic-circuits#sequential-divider) introduced in the lecture 04.

<details>

<summary>Why "strictly" follow here?</summary>

At the start, we find out that as per RISC-V register is 32-bit long, that means our **dividend** is **always** 32-bit long. So, we think that we don't need to align the **divisor** into 64-bit (2x32=64). Instead, we can just don't do any alignment of the divisor and use its original value.

However, this is proved to be a bad decision. As another purpose of aligning the divisor is to get the **first correct** subtraction from the remainder. In other words, **start at a correct starting position**. If you just use the original divisor, the first subtraction from the remainder (which is just your dividend at start) using the original divisor will lead you to the **wrong result** of division in the end.

This doesn't mean the finding that there is only 32-bit dividend in the RISC-V `div`-related instruction is useless. Maybe we can have **efficient** divide algorithm based on that?

</details>

### Implementation Details

As you may notice, the [sequential divider](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/arithmetic-circuits#sequential-divider) we have introduced in Lec 04 by right supports only **unsigned division**. However, we can only do some minor changes to make it support signed by considering the following cases. (in other words, find patterns from the following cases)

<table><thead><tr><th data-type="number">Dividend</th><th data-type="number">Divisor</th><th data-type="number">Quotient</th><th data-type="number">Remainder</th></tr></thead><tbody><tr><td>5</td><td>-3</td><td>-1</td><td>2</td></tr><tr><td>5</td><td>3</td><td>1</td><td>2</td></tr><tr><td>-5</td><td>3</td><td>-1</td><td>-2</td></tr><tr><td>-5</td><td>-3</td><td>1</td><td>-2</td></tr></tbody></table>

From the table above, we can clearly see that to do the **signed division**, we can just convert everything to **unsigned** first, then determine the sign of the quotient and the remainder.

* Quotient is **positive** if and only if the sign of the dividend and divisor are **the same**. Otherwise, quotient is **negative**.
* The sign of the remainder **follows** the sign of the **dividend**.

And to test whether our signed/unsigned multiplication/division works normally or not, we craft the following test cases:

{% tabs %}
{% tab title="Unsigned Mulciplication" %}
| Test | Operand A  | Operand B  | Expected 64-bit Product | Notes            |
| :--: | ---------- | ---------- | ----------------------- | ---------------- |
|   1  | 0x00000000 | 0x00000000 | 0x0000000000000000      | Zero × Zero      |
|   2  | 0x00000001 | 0xFFFFFFFF | 0x00000000FFFFFFFF      | Identity × Max   |
|   3  | 0xFFFFFFFF | 0xFFFFFFFF | 0xFFFFFFFE00000001      | Max × Max        |
|   4  | 0x0000FFFF | 0x0000FFFF | 0x00000000FFFE0001      | 16-bit check     |
|   5  | 0x7FFFFFFF | 0x00000002 | 0x00000000FFFFFFFE      | Near sign bit    |
|   6  | 0x80000000 | 0x00000002 | 0x0000000100000000      | 2³¹ × 2          |
|   7  | 0xFFFFFFFF | 0x00000002 | 0x00000001FFFFFFFE      | Max × 2          |
|   8  | 0xAAAAAAAA | 0x00000002 | 0x0000000155555554      | Alternating bits |
|   9  | 0x12345678 | 0x9ABCDEF0 | 0x0B00EA4E242D2080      | Random case      |
|  10  | 0x0000ABCD | 0x00001234 | 0x000000000C35F3F4      | Mid-range test   |
{% endtab %}

{% tab title="Signed Multiplication" %}
| Test | Operand A  | Operand B  | Expected 64-bit Product (signed) | Notes                |
| :--: | ---------- | ---------- | -------------------------------- | -------------------- |
|   1  | 0x00000000 | 0x00000000 | 0x0000000000000000               | Zero                 |
|   2  | 0x00000001 | 0xFFFFFFFF | 0xFFFFFFFFFFFFFFFF               | +1 × −1 = −1         |
|   3  | 0xFFFFFFFF | 0xFFFFFFFF | 0x0000000000000001               | −1 × −1 = +1         |
|   4  | 0x7FFFFFFF | 0x00000002 | 0x00000000FFFFFFFE               | (2³¹−1) × 2          |
|   5  | 0x80000000 | 0x00000002 | 0xFFFFFFFF00000000               | −2³¹ × 2             |
|   6  | 0x80000000 | 0xFFFFFFFF | 0x0000000080000000               | −2³¹ × −1 (overflow) |
|   7  | 0x00000010 | 0xFFFFFFF0 | 0xFFFFFFFFFFFFFF00               | 16 × −16 = −256      |
|   8  | 0x7FFFFFFF | 0xFFFFFFFF | 0xFFFFFFFF80000001               | (2³¹−1) × −1         |
|   9  | 0xFFFFFFFE | 0xFFFFFFFE | 0x0000000000000004               | (−2) × (−2)          |
|  10  | 0x00001234 | 0xFFFFABCD | 0xFFFFFFFFEC2E678C               | Mixed signs          |
{% endtab %}

{% tab title="Unsigned Division" %}
| Test | Dividend   | Divisor    | Quotient   | Remainder  | Notes              |
| :--: | ---------- | ---------- | ---------- | ---------- | ------------------ |
|   1  | 0x00000000 | 0x00000001 | 0x00000000 | 0x00000000 | Zero ÷ Nonzero     |
|   2  | 0xFFFFFFFF | 0x00000001 | 0xFFFFFFFF | 0x00000000 | ÷ 1                |
|   3  | 0xFFFFFFFF | 0x00000002 | 0x7FFFFFFF | 0x00000001 | Max ÷ 2            |
|   4  | 0x80000000 | 0x00000002 | 0x40000000 | 0x00000000 | 2³¹ ÷ 2            |
|   5  | 0x00000007 | 0x00000003 | 0x00000002 | 0x00000001 | Small remainder    |
|   6  | 0xFFFFFFFF | 0x7FFFFFFF | 0x00000002 | 0x00000001 | Large divisor      |
|   7  | 0xFFFFFFFF | 0xFFFFFFFF | 0x00000001 | 0x00000000 | Equal operands     |
|   8  | 0x00000005 | 0x00000007 | 0x00000000 | 0x00000005 | Dividend < Divisor |
|   9  | 0x0000000A | 0x00000003 | 0x00000003 | 0x00000001 | Nonzero remainder  |
|  10  | 0xFFFFFFFE | 0xFFFFFFFD | 0x00000001 | 0x00000001 | Max-1 ÷ Max-2      |
{% endtab %}

{% tab title="Signed Division" %}
| Test | Dividend   | Divisor    | Quotient   | Remainder  | Notes                        |
| :--: | ---------- | ---------- | ---------- | ---------- | ---------------------------- |
|   1  | 0x00000000 | 0x00000001 | 0x00000000 | 0x00000000 | Zero ÷ Positive              |
|   2  | 0x00000001 | 0xFFFFFFFF | 0xFFFFFFFF | 0x00000000 | +1 ÷ −1 = −1                 |
|   3  | 0xFFFFFFFF | 0xFFFFFFFF | 0x00000001 | 0x00000000 | −1 ÷ −1 = 1                  |
|   4  | 0x7FFFFFFF | 0x00000002 | 0x3FFFFFFF | 0x00000001 | Positive half division       |
|   5  | 0x80000000 | 0xFFFFFFFF | 0x80000000 | 0x00000000 | Overflow case (−2³¹ ÷ −1)    |
|   6  | 0x80000000 | 0x00000002 | 0xC0000000 | 0x00000000 | Negative ÷ Positive          |
|   7  | 0x00000010 | 0xFFFFFFF0 | 0xFFFFFFFF | 0x00000000 | 16 ÷ −16 = −1                |
|   8  | 0xFFFFFFF6 | 0x00000005 | 0xFFFFFFFE | 0xFFFFFFFC | −10 ÷ 5 = −2 r 0             |
|   9  | 0x12345678 | 0x0000F00F | 0x00000135 | 0x000026B5 | Random signed                |
|  10  | 0x80000000 | 0x00000003 | 0xD5555555 | 0xFFFFFFFD | −2³¹ ÷ 3 = floor-toward-zero |
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
We didn't include the case when divisor is 0 because it is specified clearly that the divisor won't be 0 in the lab manual.
{% endhint %}

## Task 2: Incorporate MCycle into our CPU

The whole idea here is that our processor is still **single cycle**!

### Implementation Details
