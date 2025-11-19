# Memory Systems

> This part is almost the same as NUS CG3207 Lec 07 — Memory Systems Principles. So, I will combine the information together here.

## Introduction

Computer system performance depends on **both** the memory system **and** the processor microarchitecture. But processor speed has increased at a **faster rate** than memory speeds. DRAM memories are currently **10 to 100 times slower** than processors. This is the bottleneck!

The processor communicates with the memory system over a **memory interface**. The figure below shows the **simple memory interface** used in our **multicycle RISC-V processor**. The processor sends an **address** over the **Address bus** to the memory system. For a **read**, `MemWrite` is 0 and the memory returns the **data** on the `ReadData` bus. For a **write**, `MemWrite` is 1 and the processor sends **data** to memory on the **WriteData bus**.

<figure><img src="../.gitbook/assets/memory-interface.png" alt="" width="375"><figcaption><p>The memory interface</p></figcaption></figure>

{% hint style="success" %}
`WriteData`, `Address` and `MemWrite` are signals coming out of the processor. `ReadData` is the signal coming into the processor.
{% endhint %}

To solve this bottleneck, we use the [two principles of locality](memory-systems.md#two-principles-of-locality) to build a memory hierarchy shown as follows.

<figure><img src="../.gitbook/assets/memory-hierarchy.png" alt="" width="563"><figcaption></figcaption></figure>

The processor first seeks data in a **small but fast cache** that is usually located **on the same chip**. If the data is **not available in the cache**, the processor then looks in **main memory**. If the data is **not there either**, the processor fetches the data from **virtual memory** on the **large but slow hard disk**. Using this memory hierarchy, our memory systems can quickly access the most commonly used data while still having the capacity to store large amounts of data.

Within this memory hierarchy, there are three levels

{% stepper %}
{% step %}
#### Cache

Computers store the **most commonly used instructions and data** in a **faster but smaller memory**, called a **cache**. This is also the **first level** of our memory hierarchy. The cache is usually built out of **SRAM** on the **same chip** as the processor. The **cache speed** is **comparable to the processor speed** because **SRAM is inherently faster than DRAM** and because the **on-chip memory** eliminates **lengthy delays** caused by traveling to and from a separate chip.

1. **Cache hit**: If the processor requests data that is **available in the cache**, it is returned **quickly**. This is called a **cache hit**.
2. **Cache miss**: Otherwise, the processor retrieves the data from **main memory (DRAM)**. This is called a **cache miss**.
{% endstep %}

{% step %}
#### Main Memory

Computer memory, which is our **second level** of the memory hierarchy, is generally built from DRAM chips. Unfortunately, **DRAM speed** has improved by only about **7% per year**, whereas **processor performance** has improved at a rate of **25% to 50% per year**, as shown in the figure below.

<figure><img src="../.gitbook/assets/diverging-processor-memory-performance.png" alt="" width="563"><figcaption><p>Diverging processor and memory performance</p></figcaption></figure>

{% hint style="warning" %}
#### Memory speed

Remember that speed is characterized by both **latency** and **throughput**.

* **Memory latency** is the time to access the **first byte** of information.
* **Throughput** is the number of bytes per second that can be delivered.

Main memories have good throughput but long latency.
{% endhint %}
{% endstep %}

{% step %}
#### Hard Drive

The **third level** of the memory hierarchy is the **hard drive**. The hard drive has evolved from HDD to NAND Flash devices, like SSD.

The hard drive provides an **illusion of more capacity** than actually exists in the main memory. It is thus called **virtual memory**. Main memory, also called **physical memory**, holds a **subset** of the **virtual memory**. Hence, the main memory can be viewed as a **cache** for the most commonly used data from the **hard drive**.
{% endstep %}
{% endstepper %}

In summary, the figure below illustrates this capacity and speed trade-off in the memory hierarchy and lists typical costs, access times, and bandwidth in 2021 technology. As access time decreases, speed increases.

<figure><img src="../.gitbook/assets/memory-hierarchy-components.png" alt=""><figcaption><p>Memory hierarchy components, with typical characteristics in 2021</p></figcaption></figure>

<details>

<summary>Two principles of locality</summary>

1. **Temporal locality**: If we have used a piece of data recently, we are likely to use it again soon.
2. **Spatial locality**: When we use one particular piece of data, we are likely to be interested in other pieces of data in the same area.

</details>

## Memory System Performance Analysis

There are three memory system performance metrics:

{% stepper %}
{% step %}
#### Miss Rate

$$
\text{Miss Rate} = \frac{\text{Number of misses}}{\text{Number of total memory accesses}} = 1 - \text{Hit Rate}
$$
{% endstep %}

{% step %}
#### Hit Rate

$$
\text{Hit Rate}  = \frac{\text{Number of hits}}{\text{Number of total memory accesses}}  = 1 - \text{Miss Rate}
$$
{% endstep %}

{% step %}
#### Average Memory Access Time (AMAT)

**Average memory access time (AMAT)** is the average time a processor must wait for memory per load or store instruction. It is calculated as follows:

$$
\text{AMAT} = t_{\text{cache}} + \text{MR}_{\text{cache}} \left( t_{\text{MM}} + \text{MR}_{\text{MM}} t_{\text{VM}} \right)
$$

where $$t_{\text{cache}}$$, $$t_{\text{MM}}$$, and $$t_{\text{VM}}$$ are the **access times** of the **cache**, **main memory**, and **virtual memory**, and $$\text{MR}_{\text{cache}}$$ and $$\text{MR}_{\text{MM}}$$ are the **cache** and **main memory miss rates**, respectively.

<details>

<summary>Self Diagnostics Question</summary>

Calculate the Average Memory Access Time (AMAT) for a computer system with a two-level memory hierarchy, given the following specifications:

* Cache Access Time: 1 cycle
* Cache Miss Rate: 10%
* Main Memory Access Time: 100 cycles

***

**Sol**: The formula for AMAT is:

<p align="center"><span class="math">\text{AMAT} = \text{Hit Time} + (\text{Miss Rate} \times \text{Miss Penalty})</span></p>

Substituting the values:

<p align="center"><span class="math">\begin{aligned} \text{AMAT} &#x26;= 1 + (0.10 \times 100) \\             &#x26;= 1 + 10                  \\             &#x26;= 11~\text{cycles} \end{aligned} </span></p>

</details>
{% endstep %}
{% endstepper %}

As a word of caution, performance improvements might not always be as good as they sound. As we have seen in [NUS CG3207 Lec 01](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-01-history-technology-performance#amdahls-law), the **amdahl's law** says that the effort spent on increasing the performance of a subsystem is **worthwhile only if** the subsystem **affects a large percentage** of the **overall performance**.
