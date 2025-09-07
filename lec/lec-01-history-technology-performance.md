# Lec 01- History, Technology, Performance

## Intro

### Below your program

This famous image is called [_abstraction_ in Harris & Harris](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one#abstraction). Please go back to review the work of art before continue reading!

<figure><img src="../.gitbook/assets/cg3207-lec01-below-your-program.png" alt="" width="375"><figcaption></figcaption></figure>

Basically, here we categorize these abstractions into three categories

1. **Application software**: Written in high-level language
2. **System software**: Operating System, a.k.a. service code
   1. Handling input/output
   2. Managing memory and storage
   3. Scheduling tasks & sharing      &#x20;resources
3. **Hardware**: **Processor**, memory, I/O   &#x20;controllers

### Levels of Program Code

<figure><img src="../.gitbook/assets/cg3207-lec01-levels-of-program-code.png" alt="" width="375"><figcaption></figcaption></figure>

There are three levels when we run a high-level language program (like C) on a processor

1. **High-level language**:
   1. Level of abstraction closer to      &#x20;problem domain
   2. Provides for productivity and      &#x20;portability
2. **Assembly language**: Textual representation of   &#x20;instructions
3. **Hardware representation**:
   1. Binary digits (bits)
   2. Encoded instructions and data

### ISA

Again, go back to the Harris & Harris notes about architecture and microarchitecture [here](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one#abstraction) as it is pretty well-written!

{% hint style="success" %}
ISA = assembly language programmer's / firmware engineer's view of the processor. Microarchitecture = hardware engineer's view of the processor.
{% endhint %}

### ABI

{% stepper %}
{% step %}
**What is an ABI?**

An **ABI (Application Binary Interface)** builds on the **ISA**, which already defines what instructions the CPU understands, but goes further by specifying **how programs interact with the OS, libraries, and hardware at the binary level**.
{% endstep %}

{% step %}
**What does ABI specify?**

Usually, the ABI will specify under an ISA, what is the purpose of each register. (For example, the [RISC-V Register File](https://wenbo-notes.gitbook.io/ddca-notes/textbook/architecture/assembly-language#the-register-set) is defined by its ABI)
{% endstep %}

{% step %}
**Why is ABI important?**

With ABI, we can get **binary portability**. For example, a program compiled on one Linux x86-64 system can run on another Linux x86-64 system, because both respect the same ABI rules.

So, think of ISA as the **language (**[**words**](#user-content-fn-1)[^1] **+** [**vocabulary**](#user-content-fn-2)[^2]**)**. Then ABI is like the **etiquette and customs**:

* Not just _what words mean_, but _how you greet people, how you exchange gifts, how you behave_. If two people speak the same language but follow different customs, they’ll still miscommunicate. Similarly, two programs can be compiled for the same ISA, but if they don’t agree on the ABI, they’ll misinterpret function calls or data.
{% endstep %}
{% endstepper %}

### Todos

{% stepper %}
{% step %}
**Compare and contrast API and ABI**

In short: **API = source-level contract; ABI = binary-level contract**.

| Aspect         | API                                                                                                | ABI                                                                                                                                                                                                                                             |
| -------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Level**      | Source code level                                                                                  | Binary / machine code level                                                                                                                                                                                                                     |
| **Definition** | This is the set of public types/variables/functions that you expose from your application/library. | This is how the compiler builds an application.                                                                                                                                                                                                 |
| **Example**    | C standard library function `printf()` is an API                                                   | <p>It defines things (but is not limited to):</p><ul><li>How parameters are passed to functions (registers/stack).</li><li>Who cleans parameters from the stack (caller/callee).</li><li>Where the return value is placed for return.</li></ul> |
{% endstep %}

{% step %}
**Read up on** [**Calling Convention on RISC-V**](https://riscv.org/wp-content/uploads/2024/12/riscv-calling.pdf)

In this course, we use RISC-V RV32I. So, the **calling convention in RISC-V RV32I** makes sure that **every compiler, OS, and library agrees** on where arguments/return values go. This way, code compiled separately (say, your program + a math library) can work together at the binary level.
{% endstep %}

{% step %}
**Which are the popular ISAs?**

This is simple, nowadays, we mainly has the following ISAs

1. x86
2. ARM
3. RISC-V
4. MIPS
{% endstep %}
{% endstepper %}

## History

### Digital Hardware Market Segments

1. ASIC (application specific integrated circuit)
2. ASSP (application specific standard product)
3. FPGA (field programmable gate array)

### Application Processor Market

Application processors = Processors used in smartphones / tablets (ARMv8A / 9A instruction set)

### Moore's Law

In 1965, Intel’s Gordon Moore predicted that **the&#x20;number of transistors** that can be integrated on **single chip** would **double** about **every two years**.

## Technology

### Power Trends

In CMOS IC technology,

$$
\text{Power}=\text{Capacitive Load}\times\text{Voltage}^2\times\text{Frequency}
$$

For example, suppose a new CPU has

* 85% of capacitive load of old CPU
* 15% voltage and 15% frequency reduction

The power reduction proportion is

$$
\frac{\text{P}_{\text{new}}}{\text{P}_{\text{old}}} = \frac{\text{C}_{\text{old}} \times 0.85 \times (\text{V}_{\text{old}} \times 0.85)^2 \times \text{F}_{\text{old}} \times 0.85}{\text{C}_{\text{old}} \times \text{V}_{\text{old}}^2 \times \text{F}_{\text{old}}} = 0.85^4 = 0.52
$$

But, we have reached the power wall nowadays,

* we can't reduce voltage further
* we can't remove more heat

How else can we improve performance?

### Issues and Modern Trends

To solve the power wall problem above, the modern trends are now as follows,

* Limited instruction level parallelsim (ILP), power issues
  * Cloud computing
  * Multi-core/processor systems, clusters
  * [Heterogeneous systems](#user-content-fn-3)[^3], hardware    &#x20;accelerators, hardware/software codesign, [reconfigurable computing](#user-content-fn-4)[^4]
  * Application-specific instruction-set processors: NPU, TPU, Bitcoin mining, etc.
* Reaching the limits of silicon:
  * Use compound semiconductors such as GaN, InP, etc.
* The communication bottleneck — within and between chips: the data transfer speed between processor and memory
  * SoCs, multi-chip modules
  * 3D ICs/stacking (e.g., in HBM)
  * Optical interconnects: Use light instead of electricity to communicate
  * In-memory computing
* Leakage current & short channel effects
  * Multi-gate (3D) FETs — FinFET and gate-all-around (GAA) FETs.

{% hint style="info" %}
The first level bullet points are the **problems**, the second level bullet points are the **modern trends solutions**.
{% endhint %}

### Todos

{% stepper %}
{% step %}
**Read up on chip binning**

**Chip binning** = sorting tested chips into categories (“bins”) based on their measured performance characteristics. An example is, same silicon die[^5] may become a “Core i9” if it passes high-frequency tests, or a “Core i5” if only stable at lower clocks.

This can **maximizes** [**yield**](#user-content-fn-6)[^6], so instead of throwing away chips that can’t meet the top spec, sell them in lower bins.
{% endstep %}
{% endstepper %}

## Performance

[^1]: words are instructions.

[^2]: vocabulary is the instruction set.

[^3]: Use different type of processors to do different tasks

[^4]: FPGAs

[^5]: A single, individual integrated circuit that has been cut from the wafer and contains the complete electronic circuitry for a specific function. From a "die" to "chip", **packaging** is needed.

[^6]: yield is the proportion of working dies per wafer.
