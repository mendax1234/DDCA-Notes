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

<figure><img src="../.gitbook/assets/memory-hierarchy.png" alt="" width="563"><figcaption><p>A typical memory hierarchy</p></figcaption></figure>

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

<details>

<summary>Memory Speed</summary>

Remember that speed is characterized by both **latency** and **throughput**.

* **Memory latency** is the time to access the **first byte** of information.
* **Throughput** is the number of bytes per second that can be delivered.

Main memories have good throughput but long latency.

</details>
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

<summary>Self Diagnostic Question</summary>

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

## Cache

A **cache** holds **commonly used memory data**. The number of data words that it can hold is called the **capacity**, $$C$$. Because the **capacity of the cache** is **smaller than that of main memory**, the computer system designer must choose what **subset** of the **main memory** is kept in the cache.

As we explain in the following sections, **caches** are specified by their **capacity** ($$C$$), **number of sets** ($$S$$), **block size** ($$b$$), **number of blocks** ($$B$$), and **degree of associativity** ($$N$$).

{% hint style="warning" %}
Keep in mind that the spirit of cache is the [two principles of locality](memory-systems.md#two-principles-of-locality) we have introduced above, which is the inherent spatial and temporal locality of data accesses in most applications.
{% endhint %}

### Data

Because it is **impossible to predict the future with perfect accuracy**, the **cache** must **guess** what data will be needed based on the **past pattern** of **memory accesses**. This pattern is guided by the [two principles of locality](memory-systems.md#two-principles-of-locality):

1. By the **temporal locality**, when the processor **loads or stores data** that is **not in the cache**, the data is **copied from main memory into the cache**. **Subsequent requests** for that data then **hit in the cache**.
2. By the **spatial locality**, when the **cache** fetches one word from memory, it may also fetch several **adjacent words**. This group of words is called a **cache block** or **cache line**. The number of words in the cache block, $$b$$, is called the **block size**. A cache of **capacity** $$C$$ contains $$B = C/b$$  **blocks**.

### Mapping

A **cache** is organized into **S sets**, each of which holds one or more **blocks** of data. The relationship between the address of data in **main memory** and the location of that data in the cache is called the **mapping**. Each **memory address** maps to **exactly one set** in the cache. Some of the **address bits** are used to determine which **cache set** contains the data. If the set contains more than one block, the data may be kept in **any** of the blocks in the set.

**Caches** are categorized based on the **number of blocks** in a set.

1. In a **direct-mapped cache**, each set contains **exactly one block**, so the cache has **S = B sets**. Thus, a particular **main memory address** maps to a **unique block** in the cache.
2. In an **N-way set associative cache**, each set contains **N blocks**. The address still maps to a **unique set**, with **S = B/N sets**. But the data from that address can go in **any of the N blocks** in that set.
3. A **fully associative cache** has only **S = 1 set**. Data can go in **any** of the **B blocks** in the set. Hence, a **fully associative cache** is another name for a **B-way set associative cache**.

To illustrate these **cache organizations**, we will consider a **RISC-V memory system** with **32-bit addresses** and **32-bit words**. The memory is **byte-addressable**, and each word is **four bytes**, so the memory consists of $$2^{30}$$ **words** aligned on word boundaries. We analyze caches with an **eight-word capacity** ($$C$$) for the sake of simplicity. We begin with a **one-word block size** ($$b$$), then generalize later to **larger blocks**.

#### Direct Mapped Cache

A **direct-mapped cache** has **one block** in each set, so it is organized into **S = B sets**. This mapping is illustrated in the figure below for a direct mapped cache with a capacity of eight words and a block size of one word.

<figure><img src="../.gitbook/assets/direct-mapped-cache.png" alt="" width="563"><figcaption><p>Mapping of main memory to a direct mapped cache</p></figcaption></figure>

The **cache** has **eight sets**, each of which contains a **one-word block**. The bottom two bits of the address are always 00, because they are **word-aligned**. The next $$\log_2 8 = 3$$ bits indicate the **set** onto which the memory address maps. Thus, the data at addresses `0x00000004`, `0x00000024`, …, `0xFFFFFFE4` all map to **set 1**, as shown in blue. Likewise, data at addresses `0x00000010`, …, `0xFFFFFFF0` all map to **set 4**, and so forth. Each **main memory address** maps to **exactly one set** in the cache.

{% stepper %}
{% step %}
#### Tag Bit

Because many addresses map to a single set, the cache must also keep track of the address of the data actually contained in each set. This is achieved by the **tag bit** and it indicates which of the many possible addresses is held in that set.
{% endstep %}

{% step %}
#### Cache Field

In our previous examples, the **two least significant bits** of the **32-bit address** are called the **byte offset** because they indicate the **byte within the word**. The next **three bits** are called the **set bits** because they indicate the **set** to which the address maps. (In general, the number of set bits is $$\log_2 S$$.) The remaining **27 tag bits** indicate the **memory address** of the data stored in a given cache set. The following figure shows the **cache fields** for address `0xFFFFFFE4`. It maps to **set 1** and its **tag** is all 1’s.

<figure><img src="../.gitbook/assets/cache-field-example.png" alt="" width="472"><figcaption></figcaption></figure>

<details>

<summary>Self Diagnostic Quiz</summary>

Find the number of set and tag bits for a direct-mapped cache with 1024 (2¹⁰) sets and a one-word block size. The address size is 32 bits.

***

**Solution:** A cache with 2¹⁰ sets requires log₂(2¹⁰) = 10 set bits. The two least significant bits of the address are the byte offset, and the remaining 32 − 10 − 2 = 20 bits form the tag.

</details>
{% endstep %}

{% step %}
#### Valid Bit

Sometimes, such as when the computer first starts up, the **cache sets** contain **no data** at all. The cache uses a **valid bit** for each set to indicate whether the set holds **meaningful data**. If the **valid bit** is **0**, the contents are **are meaningless**.
{% endstep %}

{% step %}
#### Hardware Implementation

The following figure shows the hardware for the direct mapped cache of the above figure. The **cache** is constructed as an **eight-entry SRAM**. Each **entry** (or **set**) contains one line consisting of **32 bits of data**, **27 bits of tag**, and **1 valid bit**. The cache is accessed using the **32-bit address**.

<figure><img src="../.gitbook/assets/direct-mapped-cache-hardware.png" alt="" width="563"><figcaption><p>Direct mapped cache with 8 sets</p></figcaption></figure>

A **load instruction** reads the specified **entry** from the **cache** and checks the **tag** and **valid bits**. If the **tag** matches the most significant **27 bits** of the requested address and the **valid bit** is **1**, the cache **hits** and the **data** is returned to the processor. Otherwise, the cache **misses** and the memory system must fetch the data from **main memory**.
{% endstep %}

{% step %}
#### Conflict

When two **recently accessed addresses** map to the **same cache block**, a **conflict** occurs, and the **most recently accessed address** **evicts** the previous one from the block.

**Direct-mapped caches** have **only one block** in each set, so two addresses that map to the **same set** always cause a **conflict**.
{% endstep %}
{% endstepper %}

#### Multiway Set Associative Cache

An **N-way set associative cache** reduces **conflicts** by providing **N blocks** in each set where data mapping to that set might be found. Each **memory address** still maps to a **specific set**, but it can map to **any one of the N blocks** in the set. **N** is also called the **degree of associativity** of the cache.

{% hint style="success" %}
A **direct mapped cache** is another name for a **one-way** set associative cache.
{% endhint %}

The following figure shows the hardware for a **C = 8-word**, **N = 2-way set associative cache**. The cache now has only **S = 4 sets** rather than 8. Thus, only **log₂4 = 2 set bits** rather than 3 are used to select the set. The **tag** increases from **27** to **28 bits**. Each set contains **two ways** (or degrees of associativity). Each **way** consists of a **data block** and the **valid** and **tag bits**.

<figure><img src="../.gitbook/assets/set-associative-cache-hardware.png" alt="" width="563"><figcaption></figcaption></figure>

The cache reads blocks from **both ways** in the selected set and checks the **tags** and **valid bits** for a **hit**. If a hit occurs in one of the ways, a multiplexer selects data from that way.

#### Fully Associative Cache

A **fully associative cache** contains a **single set** with **B ways**, where **B** is the **number of blocks**. A **memory address** can map to a block in **any** of these ways. A **fully associative cache** is another name for a **B-way set associative cache** with **one set**.

The following figure shows the **SRAM array** of a **fully associative cache** with **eight blocks**. Upon a data request, **eight tag comparisons** (not shown) must be made because the data could be in **any block**. Similarly, an **8:1 multiplexer** chooses the proper data if a **hit** occurs.

<figure><img src="../.gitbook/assets/fully-associative-cache-hardware.png" alt=""><figcaption></figcaption></figure>

**Fully associative caches** tend to have the **fewest conflict misses** for a given cache capacity, but they require **more hardware** for additional **tag comparisons**. They are best suited to **relatively small caches** because of the **large number of comparators**.

#### Block Size

In our previous examples, we only take advantage of the **temporal locality** because the block size was one word. To exploit **spatial locality**, a cache uses larger blocks to hold several consecutive words so that when a miss occurs and the word is fetched into the cache, the adjacent words in the block are also fetched. Therefore, subsequent accesses are more likely to hit because of spatial locality.

{% stepper %}
{% step %}
#### Miss Penalty

The time required to load the **missing block** into the **cache** is called the **miss penalty**.
{% endstep %}

{% step %}
#### Block Offset Bits

The following figure shows the hardware for a **C = 8-word direct-mapped cache** with a **b = 4-word block size**. The cache now has only **B = C/b = 2 blocks**. A **direct-mapped cache** has **one block** in each set, so this cache is organized as **two sets**. Thus, only **log₂2 = 1 bit** is used to select the set. A **multiplexer** is now needed to select the **word within the block**. The multiplexer is controlled by the **log₂4 = 2 block offset bits** of the address. The **most significant 27 address bits** form the **tag**. Only **one tag** is needed for the entire block, because the words in the block are at **consecutive addresses**.

<figure><img src="../.gitbook/assets/direct-mapped-cache-with-4-word-block-size.png" alt=""><figcaption></figcaption></figure>

The following figure shows the **cache fields** for address `0x8000009C` when it maps to the **direct-mapped cache** of the above figure. The **byte offset bits** are always 0 for word accesses. The next log₂b = 2 **block offset bits** indicate the **word within the block** and the next **bit** indicates the **set**. The remaining **27 bits** are the **tag**. Therefore, word `0x8000009C` maps to **set 1**, **word 3** in the cache.

<figure><img src="../.gitbook/assets/cache-field-example-2.png" alt="" width="532"><figcaption></figcaption></figure>

The principle of using larger block sizes to exploit spatial locality also applies to **associative caches**.
{% endstep %}
{% endstepper %}

#### Summary

**Caches** are organized as **two-dimensional arrays**. The **rows** are called **sets**, and the **columns** are called **ways**. Each **entry** in the array consists of a **data block** and its associated **valid** and **tag bits**. **Caches** are characterized by

* **capacity** $$C$$
* **block size** $$b$$ (and **number of blocks**, $$B = C/b$$)
* **number of blocks in a set /** degree of associativity ($$N$$)

The following table summarizes the various **cache organizations**. Each **address** in memory maps to **only one set** but can be stored in **any of the ways**.

<figure><img src="../.gitbook/assets/cache-organization.png" alt="" width="563"><figcaption></figcaption></figure>

**Cache capacity**, **associativity**, **set size**, and **block size** are typically **powers of two**. This makes the **cache fields** (**tag**, **set**, and **block offset bits**) subsets of the **address bits**.

**Increasing the associativity** $$N$$ usually **reduces the miss rate** caused by **conflicts**. But higher associativity requires **more tag comparators**. **Increasing the block size** $$b$$ takes advantage of **spatial locality** to **reduce the miss rate**. However, it **decreases the number of sets** in a fixed-sized cache and, therefore, could lead to **more conflicts**. It also **increases the miss penalty**.

### Replacement

In the cache, when a set is full, the block must be kicked out to be replaced with a new block,

* In a direct-mapped cache, each address maps to a unique block and set, so if a set is full when new data must be loaded, the block in that set is replaced with the new data.
* In set-associative and fully associative caches, the cache must choose which block to evict when a cache set is full.

There are three replacement algorithms, the first two are based on the history, which are also practical, while the third is optimal but as it needs to predict the future, this is not practical.

#### FIFO

FIFO stands for First-In-First-Out. In this alogrithm, the **oldest block** will be replaced. For example, assuming that we are working with a fully associative cache which has 4 blocks.

<figure><img src="../.gitbook/assets/cg3207-lec07-fifo-replacement-example.png" alt=""><figcaption></figcaption></figure>

FIFO works well if the access follows a sequential pattern.

#### Leat Recently Use (LRU)

The principle of temporal locality suggests that the best choice is to evict the least recently used block because it is least likely to be used again soon. Hence, most associative caches have a **least recently used (LRU)** replacement policy.

In a **two-way set-associative cache**, a **use bit (U)** indicates which way within the set was **least recently used**; each access flips U to point to the other way. For **set-associative caches** with more than two ways, exact tracking of the **least recently used** way is complex, so the ways are typically divided into two groups and U identifies which **group** was least recently used. On replacement, the new block evicts a **random** block from the least recently used group. This approximation, known as **pseudo-LRU**, is sufficiently effective in practice.

<details>

<summary>Self Diagnostic Quiz</summary>

Show the contents of an eight-word two-way set-associative cache after executing the following code, assuming LRU replacement, a block size of one word, and an initially empty cache.

{% code lineNumbers="true" %}
```riscv
addi t0, zero, 0
lw   s1, 0x4(t0)
lw   s2, 0x24(t0)
lw   s3, 0x54(t0)
```
{% endcode %}

**Solution**: The first two instructions load data from memory addresses 0x4 and 0x24 into set 1 of the cache, shown in the figure below, where U = 0 indicates that data in way 0 was the least recently used.

<figure><img src="../.gitbook/assets/lru-example-1.png" alt="" width="563"><figcaption></figcaption></figure>

The next memory access, to address 0x54, also maps to set 1 and replaces the least recently used data in way 0, as shown in the figure below, after which the use bit U is set to 1 to indicate that data in way 1 was the least recently used.

<figure><img src="../.gitbook/assets/lru-example-2.png" alt="" width="563"><figcaption></figcaption></figure>

</details>

### Advanced Cache Design

In this section, we will introduce the multiple-level caches, which are used quite often in modern systems. Then we will discuss the performance of a two-level caching system and examines how block size, associativity, and cache capacity affect **miss rate**. Finally, we will introduce how caches handle stores, or writes, by using a write-through or write-back policy.

#### Multiple-Level Caches

**Large caches** are beneficial because they are more likely to contain the data of interest and thus exhibit **lower miss rates**; however, **large caches** are inherently **slower** than small ones. To balance these trade-offs, modern systems employ **multiple levels of caches**, as illustrated in the figure below.

<figure><img src="../.gitbook/assets/multiple-level-caches.png" alt="" width="301"><figcaption></figcaption></figure>

The **first-level (L1) cache** is designed to be small enough to achieve a **one- or two-cycle access time**, while the **second-level (L2) cache**, also constructed from **SRAM**, is significantly **larger** — and consequently **slower** — than the **L1 cache**.

#### Reducing Miss Rate

The misses can be classified as **compulsory**, **capacity**, and **conflict**.

* **Compulsory Miss**: This happens because the block must be retrieved from memory the first time it is accessed, regardless of the cache design
* **Capacity Miss**: This occurs when the cache is too small to hold all the data that the program is actively using at the same time.
* **Conflict Miss**: This arises in **set-associative** or **direct-mapped** caches when multiple addresses map to the same **set**, causing needed blocks to be prematurely evicted even though space may be available in other sets.

Changing cache parameters can affect one or more types of cache miss. For example, increasing **cache capacity** can reduce both **capacity misses** and **conflict misses**, but it has no effect on **compulsory misses**. In contrast, increasing **block size** can reduce **compulsory misses** by exploiting **spatial locality** (bringing in more adjacent data with each miss), yet it may increase **conflict misses** because a larger block size means fewer sets in the cache, causing more addresses to map to the same set and compete for space.

{% stepper %}
{% step %}
#### Cache Size vs. Miss Rate

<figure><img src="../.gitbook/assets/cache-size-vs-miss-rate.png" alt="" width="563"><figcaption></figcaption></figure>

As expected, when cache size increases, capacity misses decrease. Increased associativity, especially for small caches, decreases the number of conflict misses shown along the top of the curve.
{% endstep %}

{% step %}
#### Block Size vs.Miss Rate

<figure><img src="../.gitbook/assets/cache-size-vs-block-size.png" alt="" width="563"><figcaption></figcaption></figure>

For small caches, such as a 4 KiB cache, increasing the **block size** beyond 64 bytes raises the **miss rate** due to fewer sets and higher **conflict misses**. In larger caches, increasing the block size beyond 64 bytes typically does not further reduce (and may not affect) the miss rate, as capacity and compulsory misses dominate. Nevertheless, **large block sizes** can still increase overall execution time because of the higher **miss penalty —** the longer time required to transfer the larger block from main memory on a miss.
{% endstep %}
{% endstepper %}

#### Write Policy

Caches are classified as either **write-through** or **write-back**.

* In a **write-through** cache, every write to a cache block is immediately written to main memory as well.
* In a **write-back** cache, each cache block has a **dirty bit (D)** that is set to 1 when the block is modified and remains 0 otherwise; a dirty block is written back to main memory only when it is evicted from the cache.

Although a write-through cache needs no dirty bit, it generally performs far more main-memory writes than a write-back cache. Because main-memory access time is very large, modern caches are almost always **write-back**.

## Vitural Memory

Most modern computer systems use a hard drive made of **magnetic** or **solid-state storage** as the lowest level in the **memory hierarchy**. The objective of adding a **hard drive** to the memory hierarchy is to inexpensively give the illusion of a very large memory while still providing the speed of faster memory for most accesses. A computer with only **16 GiB of DRAM**, for example, could effectively provide **128 GiB** of memory using the hard drive.

* This larger **128 GiB** memory is called **virtual memory**,
* and the smaller **16 GiB** main memory is called **physical memory**.

We will use the term **physical memory** to refer to **main memory** throughout this section.

<details>

<summary>iB vs B</summary>

The "i" stands for binary.

* KB = Kilobyte (standard metric prefix)
* KiB = Kilo binary byte (binary prefix)

The following table summarizes the difference between 4KiB and 4KB.

| Unit  | Name     | Math Base           | Value in Bytes                        |
| ----- | -------- | ------------------- | ------------------------------------- |
| 4 KB  | Kilobyte | Decimal ($$10^3$$)  | $$4 \times 1000 = 4000~\text{bytes}$$ |
| 4 KiB | Kibibyte | Binary ($$2^{10}$$) | $$4 \times 1024 = 4096~\text{bytes}$$ |

</details>

Programs can access data anywhere in **virtual memory**, so they must use **virtual addresses** that specify the location in virtual memory. The **physical memory** holds a subset of the most recently accessed virtual memory. In this way, physical memory acts as a **cache** for virtual memory. Thus, most accesses **hit** in physical memory at the speed of **DRAM**, yet the program enjoys the capacity of the larger virtual memory.

{% hint style="success" %}
Think of physical memory as a **cache** (later you will see it's actually a [fully associative cache](memory-systems.md#fully-associative-cache)) for the virtual memory!&#x20;
{% endhint %}

Given that, we can find the similarities between the **virtual memory** and [**cache**](memory-systems.md#cache-1) we have discussed above. The following table summarizes the analogous terms.

<figure><img src="../.gitbook/assets/virtual-memory-vs-cache.png" alt="" width="410"><figcaption><p>Analogous cache and virtual memory terms</p></figcaption></figure>

**Virtual memory** is divided into **virtual pages**, typically **4 KiB** in size. **Physical memory** is likewise divided into **physical pages** (also called **page frames**) of the same size. A **virtual page** may be located in physical memory (**DRAM**) or on the **hard drive**. The following figure shows a virtual memory that is larger than physical memory.

<figure><img src="../.gitbook/assets/virtual-and-physical-pages.png" alt="" width="563"><figcaption><p>Virtual and physical pages</p></figcaption></figure>

The rectangles indicate **pages**. Some **virtual pages** are present in **physical memory**, and some are located on the **hard drive**. The process of determining the [**physical address**](#user-content-fn-1)[^1] from the **virtual address** is called **address translation**. If the processor attempts to access a virtual address that is not in physical memory, a **page fault** occurs and the **operating system (OS)** loads the page from the hard drive into physical memory.

{% hint style="warning" %}
The processor can only access **physical memory** (DRAM), not the hard disk directly. If a virtual address maps to a page stored on the disk (Valid Bit = 0), the hardware raises a **Page Fault** exception. The Operating System handles this by retrieving the page from the disk into DRAM and updating the page table, allowing the hardware to **retry** the instruction.
{% endhint %}

To avoid **page faults** caused by [**conflicts**](memory-systems.md#reducing-miss-rate), any **virtual page** can map to any **physical page**. In other words, **physical memory** behaves as a **fully associative cache** for **virtual memory**. In the cache part, we have seen that a [fully associative cache](memory-systems.md#fully-associative-cache) has the disadvantage of having too many **ways**, thus needing a lot of comparators to determine whether the request hits the block. A realistic **virtual memory** system also has so many **physical pages** that providing a comparator for each page would be excessively expensive. Instead, the virtual memory system uses a **page table** to perform **address translation**. A **page table** contains an **entry** for each **virtual page**, indicating its location in **physical memory** or that it is on the **hard drive**. Each load or store instruction requires a **page table** access followed by a **physical memory** access. The page table access translates the **virtual address** used by the program to a **physical address**. The physical address is then used to actually read or write the data.

The **page table** is usually so large that it is located in **physical memory**. Hence, each load or store involves two physical memory accesses: a page table access and a data access. To speed up **address translation**, a **translation lookaside buffer (TLB)** caches the most commonly used **page table entries**.

### Address Translation

In a system with **virtual memory**, programs use **virtual addresses** so that they can access a large memory. The computer must translate these virtual addresses to either find the address in **physical memory** or take a **page fault** and fetch the data from the **hard drive**.

#### Page number and Page offset

Recall that virtual memory and physical memory are divided into pages.

* The most significant bits of the virtual or physical address specify the **virtual** or **physical page number**.
* The least significant bits specify the word within the page and are called the **page offset**.

#### How address translation works

The following figure illustrates the page organization of a virtual memory system with **2 GiB** of **virtual memory** and **128 MiB** of **physical memory** divided into **4 KiB** pages.

<figure><img src="../.gitbook/assets/physical-and-virtual-pages.png" alt=""><figcaption><p>Figure 8.21 Physical and virtual pages</p></figcaption></figure>

Because the **page size** is **4 KiB** = 2¹² bytes, there are 2³¹/2¹² = 2¹⁹ **virtual pages** and 2²⁷/2¹² = 2¹⁵ **physical pages**. Thus, the **virtual page number** and **physical page number** are 19 and 15 bits, respectively. Physical memory can only hold up to 1/16th of the virtual pages at any given time. The rest of the virtual pages are kept on the **hard drive**.

The following figure illustrates the translation of a **virtual address** to a **physical address**. The least significant **12 bits** indicate the **page offset** and require no translation. The upper **19 bits** of the virtual address specify the **virtual page number (VPN)** and are translated to a **15-bit physical page number (PPN)**.

<figure><img src="../.gitbook/assets/page-translation.png" alt="" width="375"><figcaption><p>Translation from virtual address to physical address</p></figcaption></figure>

<details>

<summary>Self-Diagnostic Quiz</summary>

Find the **physical address** of **virtual address** 0x247C using the virtual memory system shown in figure 8.21.

***

**Solution.** The **12-bit page offset** (`0x47C`) requires no translation. The remaining **19 bits** of the **virtual address** give the **virtual page number**, so **virtual address** `0x247C` is found in **virtual page** `0x2`. In the figure 8.21, **virtual page** `0x2` maps to **physical page** `0x7FFF`. Thus, **virtual address** `0x247C` maps to **physical address** `0x7FFF47C`.

</details>

### Page Table

The processor uses a **page table** to translate **virtual addresses** to **physical addresses**. The **page table** contains an **entry** for each **virtual page**. This entry contains a **physical page number** and a **valid bit**. If the **valid bit** is 1, the virtual page maps to the physical page specified in the entry. Otherwise, the virtual page is found on the **hard drive**.

Because the **page table** is so large, it is stored in **physical memory**. Let us assume for now that it is stored as a contiguous array, as shown in the following figure.

<figure><img src="../.gitbook/assets/page-table-example.png" alt="" width="271"><figcaption><p>The page table for Figure 8.21</p></figcaption></figure>

This page table contains the mapping of the memory system of the above figure. The **page table** is indexed with the **virtual page number (VPN)**. For example, **entry** 5 specifies that **virtual page** 5 maps to **physical page** 1. **Entry** 6 is invalid (V = 0), so virtual page 6 is located on the **hard drive**.

To perform a load or store, the processor must first translate the **virtual address** to a **physical address** and then access the data at that physical address. The processor extracts the **virtual page number** from the virtual address and adds it to the **page table register** to find the physical address of the **page table entry**. The processor then reads this page table entry from physical memory to obtain the **physical page number**. If the **entry** is **valid**, it merges this physical page number with the **page offset** to create the physical address. Finally, it reads or writes data at this physical address. Because the **page table** is stored in **physical memory**, each load or store involves two physical memory accesses.

<details>

<summary>Self-Diagnostic Quiz</summary>

Find the physical address of virtual address 0x247C using the page table shown in the figure above.

***

**Solution.** The following figure shows the **virtual address** to **physical address** translation for **virtual address** `0x247C`.

<figure><img src="../.gitbook/assets/page-table-hardware.png" alt="" width="375"><figcaption><p>Address translation using the page table</p></figcaption></figure>

The **12-bit page offset** requires no translation. The remaining **19 bits** of the virtual address are the **virtual page number**, `0x2`, and give the index into the **page table**. The page table maps **virtual page** `0x2` to **physical page** `0x7FFF`. So, virtual address `0x247C` maps to **physical address** `0x7FFF47C`. The least significant **12 bits** are the same in both the physical and the virtual address.

</details>

### Translation Lookaside Buffer

**Virtual memory** would have a severe performance impact if it required a **page table** read on every load or store, doubling the delay of loads and stores. To solve this, n general, the processor can keep the last several **page table entries** in a small cache called a **translation lookaside buffer (TLB)**. The processor “looks aside” to find the translation in the **TLB** before having to access the **page table** in **physical memory**.

A **TLB** is organized as a **fully associative cache** and typically holds 16 to 512 **entries**. Each **TLB entry** holds a **virtual page number** and its corresponding **physical page number**. The **TLB** is accessed using the **virtual page number**. If the **TLB** **hits**, it returns the corresponding **physical page number**. Otherwise, the processor must read the **page table** in **physical memory**. The **TLB** is designed to be small enough that it can be accessed in less than one cycle.

<details>

<summary>Self-Diagnostic Quiz</summary>

**Consider** the virtual memory system of the above figure. **Use** a **two-entry TLB** or explain why a **page table** access is necessary to translate **virtual addresses** `0x247C` and `0x5FB0` to **physical addresses**. Suppose that the **TLB** currently holds valid translations of **virtual pages** `0x2` and `0x7FFFD`.

***

**Solution.** The following figure shows the **two-entry TLB** with the request for **virtual address** `0x247C`.

<figure><img src="../.gitbook/assets/tlb-example.png" alt="" width="563"><figcaption><p>Address translation using a two-entry TLB</p></figcaption></figure>

The **TLB** receives the **virtual page number** of the incoming address, `0x2`, and compares it to the **virtual page number** of each **entry**. **Entry** 0 matches and is **valid**, so the request **hits**. The translated **physical address** is the **physical page number** of the matching entry, `0x7FFF`, concatenated with the **page offset** of the virtual address. As always, the **page offset** requires no translation.

The request for **virtual address** `0x5FB0` **misses** in the **TLB**. So, the request is forwarded to the **page table** for translation.

</details>

### Memory Protection

No program should be able to access another program's memory without permission, this is called **memory protection.**

**Virtual memory** systems provide **memory protection** by giving each program its own **virtual address space**. Each program can use as much memory as it wants in that virtual address space, but only a portion of the virtual address space is in **physical memory** at any given time. Each program can use its entire **virtual address space** without having to worry about where other programs are physically located. However, a program can access only those **physical pages** that are mapped in its **page table**. In this way, a program cannot accidentally or maliciously access another program’s physical pages because they are not mapped in its **page table**.

### Replacement Policies

**Virtual memory** systems use **write-back** and an **approximate least recently used (LRU)** **replacement policy**. A **write-through** policy, where each write to **physical memory** initiates a write to the **hard drive**, would be impractical. Store instructions would operate at the speed of the hard drive instead of the speed of the processor (milliseconds instead of nanoseconds). Under the **write-back** policy, the **physical page** is written back to the hard drive only when it is **evicted** from physical memory. Writing the physical page back to the hard drive and reloading it with a different **virtual page** is called **paging**, and the hard drive in a virtual memory system is sometimes called **swap space**. The processor **pages out** one of the **least recently used** physical pages when a **page fault** occurs, then replaces that page with the missing virtual page. To support these replacement policies, each **page table entry** contains two additional **status bits**: a **dirty bit (D)** and a **use bit (U)**.

The **dirty bit** is 1 if any store instructions have changed the physical page since it was read from the hard drive. When a physical page is paged out, it needs to be written back to the hard drive only if its dirty bit is 1; otherwise, the hard drive already holds an exact copy of the page.

The **use bit** is 1 if the physical page has been accessed recently. As in a cache system, exact **LRU replacement** would be impractically complicated. Instead, the **OS** approximates **LRU replacement** by periodically resetting all of the use bits in the **page table**. When a page is accessed, its use bit is set to 1. Upon a page fault, the OS finds a page with U = 0 to page out of physical memory. Thus, it does not necessarily replace the **least recently used** page, just one of the least recently used pages.

## Put it together

To put the virtual memory and cache together. We have the following diagram:

<figure><img src="../.gitbook/assets/memory-put-it-together.png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
The **Page** in the virtual address and the index `P` are equivalent to our **Virtual Page Number**. The **Frame** in the physical address is equivalent to our **Physical Page Number**.
{% endhint %}

[^1]: **Physical address** means only the address in the physical memory (main memory)!
