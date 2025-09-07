# From Zero To One

In this textbook,

1. we begin with digital logic gates that accept 1's and 0's as inputs and produce 1's and 0's as outputs.
2. We then explore how to combine logic gates into more complicated modules such as adders and memories.
3. Then we shift gears to programming in assembly language, the native tongue of the microprocessor.
4. Finally, we put gates together to build a microprocessor that runs these assembly language programs.

## The Art of Managing Complexity

One of the characteristics that separates an engineer or computer scientist from a layerperson is a systematic approach to managing complexity.

### Abstraction

The critical technique for managing complexity is _abstraction_: hiding details when they are not important. And a system can be viewed from many different levels of abstraction.

{% hint style="info" %}
This idea of abstraction has appeared in CS2030S! For example, [Data Abstraction: Type](https://wenbo-notes.gitbook.io/cs2030s-notes/lec-rec-lab-exes/lecture/lec-01-compiler-types-classes-objects#unit-2-variable-and-type-1), [Abstraction Barrier](https://wenbo-notes.gitbook.io/cs2030s-notes/lec-rec-lab-exes/lecture/lec-01-compiler-types-classes-objects#abstraction-barrier), [Functions as an Abstraction](https://wenbo-notes.gitbook.io/cs2030s-notes/lec-rec-lab-exes/lecture/lec-01-compiler-types-classes-objects#functions-as-an-abstraction).
{% endhint %}

<figure><img src="../../.gitbook/assets/level-of-abstraction.png" alt="" width="173"><figcaption></figcaption></figure>

In an electronic computer system, we can also find such abstraction:

{% stepper %}
{% step %}
**Physics**

At the lowest level of abstraction is the **physics**, which is about the motion of electrons. The behavior of electrons is described by quantum machanics and Maxwell's equations.
{% endstep %}

{% step %}
**Devices**

Our system is constructed from electronic _devices_ such as transistors. These devices have well-defined connection points called _terminals_ and can be modeled by the relationship between voltage and current as measured at each terminal. By abstracting to this **device** level, we can ignore the individual electrons.
{% endstep %}

{% step %}
**Analog Circuits**

The next level of abstraction is _analog circuits_, in which devices are assembled to create components such as amplifiers. Analog circuits input and output a continuous range of voltages.
{% endstep %}

{% step %}
**Digital Circuits**

_Digital circuits_ such as logic gates restrict the voltages to discrete ranges, which we will use to indicated 0 and 1. In logic design, we build more complex structures, such as adders or memories from digital circuits.
{% endstep %}

{% step %}
**Micro-architecture**

(See Architecture first). _Micro-architecture_ links the logic and architecture levels of abstraction. It involves combining logic elements to execute the instructions defined by the architecture/ISA.

Shortly speaking, the micro-architecture is an **implementation of the ISA/architecture**. For example, the Intel Core i7, the Intel 80486, and the AMD Athlon all implement the x86 architecture with different microarchitectures.
{% endstep %}

{% step %}
**Architecture (ISA)**

The **architecture** level describes a computer from the programmer's perspective. In other words, it's about what the programmer assumes the hardware will satisfy. It is sometimes referred to as the ISA (**I**nstruction **S**et **A**rchitecture) also, which is an interface/contract between Software and Hardware. This abstract interface between the hardware and the lowest level software encompasses all the information necessary to write a machine language program, including instructions, registers, memory access I/O, ...

For example, the Intel x86 architecture used by microprocessors in most PCs is defined by **a set of instructions** and **registers** (memory for temporarily storing variables) that the programmer is allowed to use.
{% endstep %}

{% step %}
**Operating System**

Moving into the software realm, the operating system handles low-level details such as accessing a hard drive or managing memory.
{% endstep %}

{% step %}
**Application Software**

Finally, the application software uses these facilities provided by the operating system to solve a problem for the user.
{% endstep %}
{% endstepper %}

### Discipline

_Discipline_ is the act of intentionally restricting your design choices so that you can work more productively at a higher level of abstraction.

In our context of this book, the **digital discipline** will be very important. Digital circuits use discrete voltages, whereas analog circuits use continuous voltages. Therefore, digital circuits are a **subset** of analog circuits and in some sense must be capable of less than the broader class of analog circuits. However, digital circuits are much simpler to design. By limiting ourselves to digital circuits, we can easily combine components into sophisticated systems that ultimately outperform those built from analog components in many applications.

### The Three-Y's

In addition to abstraction and discipline, designers use the three "-y's" to manage complexity: _hierarchy, modularity,_ and _regularity_.

* _Hierarchy_ involves dividing a system into **modules**, then further subdividing each of these modules until the pieces are easy to understand.
* _Modularity_ states that each module should have a well-defined function[^1] and interface[^2], so that they connect together easily without unanticipated side effects.
* _Regularity_ seeks uniformity among the modules. **Common modules** are reused many times, reducing the number of distinct modules that must be designed.

{% hint style="info" %}
These principles apply to both software and hardware systems.
{% endhint %}

## The Digital Abstraction

Digital Systems represent information with _discrete-valued variables_ — that is, variables with a finite number of distinct values.

[^1]: "Function" means this module can be used to do what.

[^2]: Think of **interface** as the _boundary_ between the inside of the module (its internal logic) and the outside world — what others can “see” and interact with, without needing to know the internal details. Technically, the **interface** specifies the _inputs, outputs,_ etc.
