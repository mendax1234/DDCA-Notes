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

<table><thead><tr><th width="85.5">Instr.</th><th width="90.5">I Mem</th><th width="94">Reg Rd</th><th width="99">ALU Op</th><th width="95">D Mem</th><th width="95.5">Reg Wr</th><th width="98.5">PC Incr</th><th width="100">Total</th></tr></thead><tbody><tr><td>DP</td><td>200</td><td>100</td><td>120</td><td></td><td>60</td><td></td><td>480</td></tr><tr><td><code>lw</code></td><td>200</td><td>100</td><td>120</td><td>200</td><td>60</td><td></td><td>680</td></tr><tr><td><code>sw</code></td><td>200</td><td>100</td><td>120</td><td>200</td><td></td><td></td><td>620</td></tr><tr><td><code>beq</code></td><td>200</td><td>100</td><td>120</td><td></td><td></td><td>75</td><td>495</td></tr><tr><td><code>j</code><del><code>al</code></del></td><td>200</td><td></td><td></td><td></td><td></td><td>75</td><td>275</td></tr></tbody></table>

{% hint style="success" %}
`PC_Incr` is done in **parallel**, when things happen in parallel, we take the worst case timing. For branch instructions and the `jal` instructions here, as it needs to fecth the instruction (200ps) and then **decide** whether to increment PC or not. Thus you will count `PC_Incr` time into the calculation.
{% endhint %}

<details>

<summary>Why all these parts take time to execute?</summary>

It is because all the Instruction Memory, Register File, ALU, Data Memory and Adders, they are implemented using a **lot of logic gates**, thus it confirm will have some propagation delay. So, it takes a certain amount of time for the signal to travel from the input to the output of that certain part.

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

**Pipelining** is to "cut" our microarchitecture[^1] into different **stages** (in our case, it is 5), and each stage will take **one clock cycle** to complete. Thus, the clock cycle time can be decreased to a large extent.

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

However, this pipeline isn't perfect, you may notice that **not every instruction** needs the **full 5 stages**. For example, `sw` doesn't need `WB`, branch and DP Reg/Imm doesn't need `Mem`, etc. Besides, there are still some **hazards** that we may encouter. (We will see later in this lec)

<details>

<summary>Is the speed-up of the pipelined processor always same as the number of stages?</summary>

Not exactly. For example, a 5-stage pipelined processor doesn't necessarily have a 5x speed-up comparing to the single-cycle processor as you can see from below,

<figure><img src="../.gitbook/assets/cg3207-lec05-speedup-of-pipelined-processor.png" alt=""><figcaption></figcaption></figure>

The reason for this is because in the pipelined processor, your clock cycle time accommodates the **slowest stage**, while in the single-cycle processor, your clock cycle time accommodates the **slowest instruction**. But the **slowest instruction** isn't composed of 5x the **slowest stage**!

</details>

## Implement a Pipelined Processor

As we have mentioned above, we need to "cut" our microarchitecture into 5 stages. To do this "cut", we are basically **adding registers** between each stage.

<figure><img src="../.gitbook/assets/cg3207-lec05-implement-cut.jpg" alt=""><figcaption></figcaption></figure>

We have 5 stages, thus we need 4 "cuts"/registers. By adding registers, we can clearly see that now our [**critical path**](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-02-digital-system-design-and-verilog#critical-path) is shortened significantly! And this practice is the **soul** of pipelining!

{% hint style="success" %}
#### Pipeline Register Naming Convention

We name the register by the **target** that it feeds the signal to. For example, the register D is named D because the here the signals coming from the **source** fetch stage will go into the **target** dec stage.
{% endhint %}

The smart you may notice that besides adding 4 registers, we also made some changes to some signals.

<figure><img src="../.gitbook/assets/cg3207-lec05-signals-change.png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
#### Delayed Signals

Several signals, like `RegWrite`, `MemtoReg` etc are delayed. Their names are also changed at each stage by adding the posfix `F, D, E, M, W` depends on which stage the signals are at.

We can think of this delay as, after we "cut" the microarchitecture, we still need to "store" the corresponding Control Unit signals and the Datapath signals to make sure that the operation done in **each stage** works correctly.
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

### Pipeline Hazards

Pipelining is not perfect, sometimes it can get into troubles. These troubles are called **pipeline hazards**. And normally, we have the following three hazards

1. [Structural Hazards](lec-05-the-pipelined-processor.md#structural-hazards)
2. [Data Hazards](lec-05-the-pipelined-processor.md#data-hazards)
3. [Control Hazards](lec-05-the-pipelined-processor.md#control-hazards)

#### Structural Hazards

**Structural harzards** happen when we attempt to use the same resource (usually it is the memory) by two different instructions at the same time.

For example, in the following diagram, at the fourth clock cycle, both the `lw` and `sw` are accessing the same memory because in real life, IROM and DMEM are both in the RAM memory.

<figure><img src="../.gitbook/assets/cg3207-lec05-structural-hazard.png" alt=""><figcaption></figcaption></figure>

#### Data Hazards

**Data harzards** happen when we attempt to use data before it is ready. For example, an instruction's source operand(s) are produced by a prior instruction still in the pipeline. This hazard is also known as _RAW_ (**read after write**) hazard or _true data dependency_ and it occurs very frequently in practice.

For example, in the following diagram, the destination register `rd` of the `add` instruction is needed by the following `sub`, `or` and `and` as a source register `rs`.

<figure><img src="../.gitbook/assets/cg3207-lec05-data-hazard.png" alt=""><figcaption></figcaption></figure>

#### Control Hazards

## Handle Pipeline Hazards

In this part, we will introduce how to handle the 3 types of pipeline harzards we have encountered above. As for the sake of this course, we will focus more on the [handling of the data hazards](lec-05-the-pipelined-processor.md#handle-data-hazards).

### Handle Structural Hazards

We can simply fix this hazard by using separate instruction and data memory.

### Handle Data Hazards

To handle data harzards, we will introduce five methods,

#### Use different clock edges

This means that we will write register file and [pipeline registers](#user-content-fn-2)[^2] at different edges of the clock.

For example, in the following diagram, we use `negedge` for register file and `posedge` for the pipeline state registers.

<figure><img src="../.gitbook/assets/cg3207-lec05-use-different-edges-clock.png" alt="" width="563"><figcaption></figcaption></figure>

Following the above convention we made, here's the updated microarchitecture of our processor

<figure><img src="../.gitbook/assets/cg3207-lec05-use-different-edges-microarch.png" alt=""><figcaption></figcaption></figure>

Using this technique to analyze the data hazard example above, we can see from the following graph that the third instruction `and` will successfully read the updated value stored in register `s8`.

<figure><img src="../.gitbook/assets/cg3207-lec05-use-differenc-edges-analysis.png" alt=""><figcaption></figcaption></figure>

Given that, we will summarize the pros and cons of this technique as follows,

* **Pros**:
  * This technique can help reduce the "troublesome" instructions from 3 to 2 in our example [above](lec-05-the-pipelined-processor.md#data-hazards).
* **Cons**:
  * It will use 2 edges from the same block and this is a practice discouraged in [CG3207.](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-02-digital-system-design-and-verilog#use-one-clock-for-the-entire-design)
  * The half cycle may not be enough to write the pipeline state register. In other words, reading the value from the register file and write it to a pipeline state register may take longer than half of a clock cycle.

#### Inserting NOPS

> In RISC-V, we can use `addi x0, x0, 0` as NOP.

The basic idea is that when there exists true data dependency between two instructions, we can insert NOPs between the these instructions to "delay" the data "transfer". So, if we have already implemented the [first technique of using different clock edges](lec-05-the-pipelined-processor.md#use-different-clock-edges), we should make sure the number of NOPs or independent useful instructions is **2** between the two instructions that have true data dependency.

{% hint style="warning" %}
This magic number 2 here is dependent on the microarchitecture of the processor. In other words, how many pipline stages you have in your processor.
{% endhint %}

For example, in the following diagram, `add` and `sub` have true data dependency. We can either insert two NOPs between `add` and `sub` or replace one of the NOPs with the independent useful instruction `orr`.

<figure><img src="../.gitbook/assets/cg3207-lec05-inserting-nops-example.png" alt=""><figcaption></figcaption></figure>

Now, we can summarize the pros and cons of this technique

* **Pros**:
  * It is easier to implement as the NOPs are inserted by the compiler during the compile time.
* **Cons**:
  * Insert NOPs will **waste code memory**.
  * As NOPs will do nothing, we won't treat them as real instructions when calculating the CPI. Thus, with more clock cycles and the same number of useful instructions, the **CPI will increase** and thus increase our CPU Time.
  * To know the number of NOPs or independant useful instructions to be inserted, the compiler needs to know the microarchitecture, like how many stages are in your pipeline processor and have you used different clock edges to write to register file and pipeline state registers, etc. This will make the code not very **portable**.

#### Data Forwarding

> This is the most challenging technique introduced in hanlding data hazards.

The basic idea in data forwarding is that the data is available on some pipeline register before it is written back to the register file (RF). So, we can take that data from where it is present, and pass it to where it is needed.

This kind of forwarding logic can be used in two stages/functional unit:

* Execution Stage, or the two sources of ALU
* Memory Stage, or the DMEM

{% stepper %}
{% step %}
#### Execution Stage

In the execution stage, we can check if the register read ([rs1E and rs2E](#user-content-fn-3)[^3]) by the instruction which is currently in Execute stage [**matches**](#user-content-fn-4)[^4] the register written ([rdM or rdW](#user-content-fn-5)[^5]) by the instruction which is currently in Memory or Writeback Stage. If so, we forward the **ready result** (usually is the ALUResult at Memory or Writeback Stage) to the Execution stage so now the input to ALU will come from M or W stage rather than the E stage register.

For example, we can see how the above forward logic works in the following graph,

<figure><img src="../.gitbook/assets/cg3207-lec05-data-forwarding-e-stage.png" alt=""><figcaption></figcaption></figure>

Suppose we have applied the [#use-different-clock-edges](lec-05-the-pipelined-processor.md#use-different-clock-edges "mention") technique, now the `add` instruction only has true data dependency with the `sub` and `or` instruction.

* In the `sub` instruction's E stage, we notice it needs `rs1E` to be the register `s8`. In the `add` instruction, its `rdM` in the M stage is register `s8` and it mathces with `rs1E`. Thus, we can forward  the ready ALUResult from `add` instruction's M stage to the E stage of the `sub` instruction.
* Similarly, in the `or` instruction's E stage, it needs `rs2E` to be the register `s8`. In the `add` instruction, its `rdW` in the W stage is register `s8` and it mathces with `rs2E`. Thus, we can forward the ready ALUResult (In W stage, it is ResultW) to the E stage of the `or` instruction.

To implement this fowarding logic, we will add **Hazard Unit** shown as follows in our microarchitecture to handle specifically handle the hazard.

<figure><img src="../.gitbook/assets/cg3207-lec05-data-forwarding-e-stage-unit.jpg" alt=""><figcaption></figcaption></figure>

`rs1/2E`, `rdM/W` and `RegWriteM/W` are easy to pass as the **input** to the Hazard Unit, but the **output** `ForwardA/BE` which are used to control the multiplexer for the ALU SrcA and SrcB should follow exactly the following three cases

{% code lineNumbers="true" %}
```verilog
// rs1E
if ((rs1E == rdM) && RegWriteM && (rdM != 0))      // Case 1
    ForwardAE = 2'b10;
else if ((rs1E == rdW) && RegWriteW && (rdW != 0)) // Case 2
    ForwardAE = 2'b01;
else                                               // Case 3
    ForwardAE = 2'b00;

// rs2E is similar, just replace rs1E with rs2E
if ((rs2E == rdM) && RegWriteM && (rdM != 0))      // Case 1
    ForwardBE = 2'b10;
else if ((rs2E == rdW) && RegWriteW && (rdW != 0)) // Case 2
    ForwardBE = 2'b01;
else                                               // Case 3
    ForwardBE = 2'b00;
```
{% endcode %}

* Condition 1 `rs1/2E == rdM/W` is about the **match**.
* Condition 2 `RegWriteM/W` ensures that the instruction (in our example, `add`) really **writes to the register**. If not (like `sw`, `branch`), there won't be any **data hazard**, thus no need to do data forwarding.
* Condition 3 `rdM/W ≠ 0` ensures that we are **not forwarding** from the `x0` register as there is no need to do so.

In total, we need 2 x 2 + 2 = 6, 5-bit comparators to do this data forwarding. `rs1/2E` each needs two comparators (2 x 2 = 4) and `rdM/W` needs another 2 but the can share result to another `Forward1/2E` signal.&#x20;
{% endstep %}

{% step %}
#### Mem-to-Mem Copy

Another situation that we should do the data forwarding is when there is a `lw` followed by a `sw` and the `sw` tries to store the value (`rs2`) that `lw` writes/loads to (`rd`).

In this case, we need to check if the register used in Memory stage by the `sw` instruction (`rs2`) matches register written by the `lw` in Writeback stage (`rd`). If so, we will forward the result.

<figure><img src="../.gitbook/assets/cg3207-lec05-data-forwading-mem-mem-copy.png" alt=""><figcaption></figcaption></figure>

And our Hazard Unit will be updated to the following,

<figure><img src="../.gitbook/assets/cg3207-lec05-data-forwarding-mem-copy-circuit.jpg" alt=""><figcaption></figcaption></figure>

By now, we have added `ForwardM` as one **output** of our Hazard Unit, `rd2M` and `rdW` are two **inputs**. And the logic to derive when to set `ForwardM` will be as follows,

```verilog
ForwardM = (rs2M == rdW) & MemWriteM & MemtoRegW & (rdW != 0)
```

* Condition 1: `rs2M == rdW` is the basic match check
* Condition 2: `MemWriteM` ensures that the instruction at the Memory stage is the `sw`. (See the [Lec 03](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-03-risc-v-isa-and-microarchitecture#support-for-link-and-jalr), same for above, as only `sw` has `MemWrite == 1`)
* Condition 3: `MemtoRegW` ensures that the instruction at the Writeback stage is the `lw`
* Condition 4: `rdW ≠ 0` ensures that the `lw` at the Writeback stage have a non-`x0` destination.
{% endstep %}
{% endstepper %}

[^1]: including the Control Unit and Datapaths

[^2]: Write to pipeline registers can be thought of as **reading** or **updating** the pipelined registers' values.

[^3]: `rs` is different from `RD`. `rs` stores the **index** of the register we want to read, e.g., if `rs1 = 1`, means we want to read from `x1` register. However, `RD` stores the **value** that we read from `rs` index register. e.g., If `rs1 = 1`, then `RD1` will store the value in register `x1`.

[^4]: "matches" just means whether `rs1E` or `rs2E` is **equal** to `rdM` or `rdW`.

[^5]: Similar as the `rs1` and `rs2`, `rd` stores the **index** of the register we want to write to. In other words, the destination register. And its correponding `WD` will store the **value** we will write to the `rd` register.
