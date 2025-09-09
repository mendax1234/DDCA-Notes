# Lec 02 - Digital System Design and Verilog

## Digital System Design

### Levels of Abstraction

Different from the [abstraction](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one#abstraction) we have seen in Harris & Harris. Here, we talk about the abstraction in digital system design.

<figure><img src="../.gitbook/assets/ cg3207-lec02-levels-of-abstraction-digital.png" alt="" width="563"><figcaption></figcaption></figure>

Let's have a detailed look on each level,

{% stepper %}
{% step %}
**Algorithm/System Level (Untimed)**

At the highest level, the design is expressed as algorithms or functional behavior without worrying about timing. For example, describing `output = (A+B) + (C+D)` in a flowchart or C-like pseudocode.

**Focus:** This part is only on the _functionality_, not on how many cycles or how it’s implemented in hardware.
{% endstep %}

{% step %}
**Register Transfer Level (RTL, Timed)**

It is the macroscopic hardware view. The system is described in terms of **data transfers between registers** and **operations performed by functional units (ALUs, multiplexers, etc.)** under clock control. It is timed, cycle-accurate, but still abstract (macroscopic). For example, the RTL Verilog Code we have written in [CG3207 Lab01](https://wenbo-notes.gitbook.io/ddca-notes/lab/lab-01-get-prepared#rtl-design) or the following simple code in Verilog

{% code lineNumbers="true" %}
```verilog
always @(posedge clk)
    output <= (A+B) + (C+D);
```
{% endcode %}

**Focus:** This is where your **macroscopic blocks** appear — ALUs, adders, multiplexers, etc.
{% endstep %}

{% step %}
**Gate Level**

It is the microscopic hardware view. RTL constructs are **synthesized into logic gates** (AND, OR, NOT, flip-flops). For example,`(A+B)` becomes a [ripple-carry adder](https://wenbo-notes.gitbook.io/ddca-notes/textbook/digital-building-blocks/arithmetic-circuits#ripple-carry-adder) built out of AND/OR/XOR gates. It is boolean equations + gates, but no transistor-level details.

**Focus:** This is your **microscopic implementation** of RTL macros.
{% endstep %}

{% step %}
**Circuit Level**

It is the actual electronic implementation of logic gates using [**CMOS Transistors**](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/cmos-transistors). For example, an inverter (`NOT gate`) is realized using one PMOS and one NMOS transistor.

**Focus:** Device-level representation, electrical properties like delay, power, capacitance are considered.
{% endstep %}

{% step %}
**Layout Level**

It is the physical representation of the circuit on silicon. Masks for fabrication are designed here. And it is the final physical placement/routing of transistors and wires.
{% endstep %}
{% endstepper %}

### Simplified FPGA/ASIC Design Flow

<figure><img src="../.gitbook/assets/cg3207-lec02-simplified-fpga-asic-design-flow.png" alt=""><figcaption></figcaption></figure>

#### Behavioural Modelling

Behavioral modeling defines the "what" of your FPGA/ASIC design — its high-level logic and algorithms— without hardware details. It’s for verifying functionality early.

* **Purpose**: Ensures algorithmic correctness via simulations.
* **Tools**:
  * Initial tests with Python, C, Java, Matlab (fast, sequential). e.g., A Python script to simulate a filter: `output = input * 0.5 if input > 0 else 0` to test logic.
  * Deeper simulations with VHDL, Verilog, SystemC (handles concurrency and timing, but no cycle accuracy).
* **Not Directly Synthesized**: Meant for validation, not hardware generation.
* **HLS Trend**: High-Level Synthesis tools (e.g., Vivado HLS) can convert behavioral code to RTL, but effectiveness varies by tool, domain, and needs manual tweaks.

Then, it will be fed into Architectural Synthesis in the design flow, making complex problems manageable through iteration.

#### Architectural Synthesis

Architectural synthesis turns a high-level functional/behavioral (acrhitectural) model into a **macroscopic** structural (microarchitectural) model for FPGAs/ASICs. It’s mostly manual but becoming more automated.

> In short, acchitectural synthesis is just to write **RTL Code**.

* **Purpose**: Converts abstract logic into a cycle-accurate, synthesizable RTL code, typically with structural and behavioral elements.
* **Output**: A block-level model where operations are **timed** and assigned to **hardware blocks**. _Example_: From `Z = (A+B) * (C+D) * E`, it creates a plan with adders and multipliers.
* **Key Steps**:
  * **Scheduling** (time): Assigns operations to clock cycles. e.g.,  `(A+B)` in cycle #1, `(C+D)` in cycle #2.
  * **Binding** (space): Maps operations to specific hardware blocks. e.g., `(A+B)` done by ALU #1, `(C+D)` by ALU #2.
  * **Flexibility**: Adders can reuse for different pairs with multiplexers. e.g., ALU #1 adds `A+B` in cycle #1, then `E+F` in cycle #2 if inputs switch.
* **Tools**: RTL synthesis infers register transfers and generates a netlist if guidelines are followed.

The following images shows the difference between behavioural modelling and architectural synthesis.

<figure><img src="../.gitbook/assets/cg3207-lec02-behavioural-vs-architectural.png" alt="" width="563"><figcaption></figcaption></figure>

Fits between Behavioral Modeling and Logic Synthesis in the design flow, balancing abstraction with hardware readiness.
