# Lec 05 - The Pipelined Processor

## Introduction

### Recap of the single cycle processor

In Lec 03, we have introduced how to implement a **single-cycled** RISC-V processor. In our design, we have noticed that different instructions will go through different **parts** of the datapath, thus causing the time taken for each instruction to complete is different.

<figure><img src="../.gitbook/assets/cg3207-lec03-support-for-link-jalr.png" alt=""><figcaption><p>A single cycle processor that we designed at Lec 03</p></figcaption></figure>

For example, let's assume

* Instruction and Data Memory needs 200ps
* ALU needs 120ps
* Adders needs 75ps
* Register File Access (reads: 100ps, writes: 60ps)

Then we will have,

| Instr.      | I Mem | Reg Rd | ALU Op | D Mem | Reg Wr | PC Incr | Total |
| ----------- | ----- | ------ | ------ | ----- | ------ | ------- | ----- |
| DP          | 200   | 100    | 120    |       | 60     |         | 480   |
| `lw`        | 200   | 100    | 120    | 200   | 60     |         | 680   |
| `sw`        | 200   | 100    | 120    | 200   |        |         | 620   |
| `beq`       | 200   | 100    | 120    |       |        | 75      | 495   |
| `j`~~`al`~~ | 200   |        |        |       |        | 75      | 275   |

{% hint style="success" %}
`PC_Incr` is done in **parallel**, when things happen in parallel, we take the worst case timing. For branch instructions and the `jal` instructions here, as it needs to fecth the instruction (200ps) and then **decide** whether to increment PC or not, thus you will count `PC_Incr` time into the calculation.
{% endhint %}

<details>

<summary>Why all these parts take time to execute?</summary>

It is because all the Instruction Memory, Register File, ALU, Data Memory and Adders, they are all implemented using a **lot of logic gate**, thus it confirm will have some propagation delay. So, it takes a certain amount of time for the signal to travel from the input to the output of that certain part.

</details>

And our single cycle processor's clock cycle time **must be** at least **larger than** the slowest instruction's execution time. (usually, it is the `lw` that takes the longest time to complete) This will create some unwanted waste in the resources, like adders.

### Make it Faster

Recall that the performance equation we introduced in [Lec 01](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-01-history-technology-performance#instruction-count-ic-and-cpi) is,

$$
\text{CPU Time} = \text{Instruction Count} \times \text{CPI} \times \text{Clock Cycle Time}
$$

Here, we cannot change IC and CPI, how can we improve the performance? The answer will be easy, that is to **decrease the** Clock Cycle Time by using **pipelining**.

#### Five Stage Pipeline

> In this course, we will use a five-stage pipeline on our processor.

**Pipeling** is to "cut" our microarchitecture[^1] into different **stages** (in our case, it is 5), and each stage takes **one clock cycle** to complete. Thus, the clock cycle time can be decreased largely.

For example, below is the image showing the fives stages of the `lw` instruction.

<figure><img src="../.gitbook/assets/cg3207-lec05-lw-five-stages.png" alt="" width="563"><figcaption></figcaption></figure>

* **Fetch**: Instruction Fetch and update PC
* **Dec:** Registers Fetch and Instruction Decode
* **Exec**: Execute DP-type; calculate memory address
* **Mem**: Read/write the data from/to the Data Memory
* **WB**: Write the result data into the register file

To see clearly how we do this "cut", we can reuse the microarchitecture in Lec 03.

<figure><img src="../.gitbook/assets/cg3207-lec05-5-cut.jpg" alt=""><figcaption></figcaption></figure>

After pipelining our processor, the clock cycle diagram to execute the instructions will look like as follows

<figure><img src="../.gitbook/assets/cg3207-lec05-clock-cycle-diagram.png" alt=""><figcaption></figcaption></figure>

We can see that after the completion of the clock cycle 5, one instruction is completed every cycle, thus CPI = 1. We also call the first 4 cycles **overhead** as nothing completes during this time.

However, this pipeline isn't perfect, you may notice that **not every instruction** needs the **full 5 stages**. For example, `sw` don't need `WB`, branch and DP Reg/Imm don't need `Mem`, etc. Besides that, there are still some **hazards** we may encouter. (We will see later in this lec)

<details>

<summary>Is the speed-up of the pipelined processor always same as the number of stages?</summary>

Not exactly. For example, a 5-stage pipelined processor doesn't necessarily have a 5x speed-up comparing to the single-cycle processor as you can see from below,

<figure><img src="../.gitbook/assets/cg3207-lec05-speedup-of-pipelined-processor.png" alt=""><figcaption></figcaption></figure>

The reason for this is because in the pipelined processor, your clock cycle time accommodates the **slowest stage**, while in the single-cycle processor, your clock cycle time accommodates the **slowest instruction**. But the **slowest instruction** isn't composed of 5x the **slowest stage**!

</details>

## Implement a Pipelined Processor

As we have mentioned above, we need to "cut" our microarchitecture into 5 stages. To do this "cut", we are actually **adding registers** between each stage.

<figure><img src="../.gitbook/assets/cg3207-lec05-implement-cut.jpg" alt=""><figcaption></figcaption></figure>

We have 5 stages, thus we need 4 "cuts"/registers. By adding registers, we can clearly see that now our [**critical path**](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-02-digital-system-design-and-verilog#critical-path) is shortened significantly! And this practice is the **soul** of pipelining!

{% hint style="success" %}
#### Pipeline Register Naming Convention

We name the register by the **target** that it feeds the signal to. For example, the register D is named D because the here the signals coming from the **source** fetch stage go into the **target** dec stage.
{% endhint %}

The smart you may notice that besides adding 4 registers, we also make some changes to some signals.

<figure><img src="../.gitbook/assets/cg3207-lec05-signals-change.png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
#### Delayed Signals

Several signals, like `RegWrite`, `MemtoReg` etc are delayed. Their names are also changed at each stage by adding the posfix `F, D, E, M, W` depends on which stage the signals are at.

We can think of this delay as, after we "cut" the microarchitecture, we still need th corresponding Control Unit Signals and the Datapath signals to make sure that the operation done in each stage works correctly.
{% endstep %}

{% step %}
#### The cut of `rdW`

In the Fetch stage, the `rd` is no longer connected directly to the instruction decoder. Instead, we delay the `rd` signal by 3 stages and connect it to `rdW`, which means after the `WD` stage, we connect the `rd` back to the register file.

This is because we are using pipeline! Suppose we have the following instructions

{% code lineNumbers="true" %}
```armasm
add x1, x2, x3
sub x4, x5, x6
```
{% endcode %}

If our `rd` is connected **directly** to the instruction decoder, the correct `rd` (`x1`) of the `add` will be overwritten by the `rd` (`x4`) of the `sub` instruction!
{% endstep %}

{% step %}
#### The multiplexer at PC

This multiplexer is added for the branch instructions and correspondingly, the `PC` signal is delayed by 2 stages so the multiplexer will choose between `PCF` and `PCE`.

This is because in the fetch stage, the PC will continuing increasing by 4 without waiting for the actually completion of one instruction. However, if our branch is taken, we should use the "old" `PC` to add to `ExtImm`.
{% endstep %}
{% endstepper %}

[^1]: including the Control Unit and Datapaths
