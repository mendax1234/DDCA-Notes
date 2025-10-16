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

After knowing this, I believe this task will become much easier!

## Task 2: Incorporate MCycle into our CPU

The whole idea here is that our processor is still **single cycle**!

### Implementation Details
