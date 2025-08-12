# From Zero To One

In this textbook,

1. we begin with digital logic gates that accept 1's and 0's as inputs and produce 1's and 0's as outputs.
2. We then explore how to combine logic gates into more complicated modules such as adders and memories.
3. Then we shift gears to programming in assembly language, the native tongue of the microprocessor.
4. Finally, we put gates together to build a microprocessor that runs these assembly language programs.

## The Art of Managing Complexity

One of the characteristics that separates an engineer or computer scientist from a layerperson is a systematic approach to managing complexity.

### Abstraction

The critical technique for managing complexity is _abstraction_: hiding details when they are not important. A system can be viewed from many different levels of abstraction.

{% hint style="info" %}
This idea of abstraction has appeared in CS2030S! Like, [Data Abstraction: Type](https://app.gitbook.com/s/vwztj22G2t5iduk4o9yh/lec-rec-lab-exes/lecture/lec-01-compiler-types-classes-objects#unit-2-variable-and-type-1 "mention"), [Abstraction Barrier](https://app.gitbook.com/s/vwztj22G2t5iduk4o9yh/lec-rec-lab-exes/lecture/lec-01-compiler-types-classes-objects#abstraction-barrier "mention") and [Functions as an Abstraction](https://app.gitbook.com/s/vwztj22G2t5iduk4o9yh/lec-rec-lab-exes/lecture/lec-01-compiler-types-classes-objects#functions-as-an-abstraction "mention").
{% endhint %}

In an electronic computer system, we can also find such abstraction. At the lowest level of abstraction is the **physics**, the motion of electrons. Our system is constructed from electronic _devices_ such as transistors. By abstracting to this device level, we can ignore the individual electrons.

### Discipline

_Discipline_ is the act of intentionally restricting yoru design choices so that you can work more productively at a higher level of abstraction.

In our context of this book, the **digital discipline** will be very important. Digital circuits use discrete voltages, whereas analog circuits use continuous voltages. Therefore, digital circuits are a subset of analog circuits and in some sense must be capable of less than the broader class of analog circuits. However, digital circuits are much simpler to design. By limiting ourselves to digital circuits, we can easily combine components into sophisticated systems that ultimately outperform those built from analog components in many applications.

### The Three-Y's

In addition to abstraction and discipline, designers use the three "-y's" to manage complexity: _hierarchy, modularity,_ and _regularity_.

* _Hierarchy_ involves dividing a system into modules, then further subdividing each of these modules until the pieces are easy to understand.
* _Modularity_ states that each module should have a well-defined function[^1] and interface[^2], so that they connect together easilty without unanticipated side effects.
* _Regularity_ seeks uniformity among the modules. Common modules are reused many times, reducing the number of distinct modules that must be designed.

{% hint style="info" %}
These principles apply to both software and hardware systems.
{% endhint %}

[^1]: "Function" means this module can be used to do what.

[^2]: Think of **interface** as the _boundary_ between the inside of the module (its internal logic) and the outside world — what others can “see” and interact with, without needing to know the internal details. Technically, the **interface** specifies the _inputs, outputs,_ etc.
