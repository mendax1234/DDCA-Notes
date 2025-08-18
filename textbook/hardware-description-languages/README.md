# Hardware Description Languages

Thus far, we have focused on designing combinational and sequential digital circuits at the schematic level. The process of finding an efficient set of logic gates to perform a given function is labor intensive and error prone, requiring manual simplification of truth tables or Boolean equations and manual translation of finite state machines (FSMs) into gates.

In 1990s, designers discovered they were far more productive if they worked at a higher level of **abstraction**, _specifying_ just the logical function and allowing a _computer-aided design_ (CAD) tool to produce the optimized gates. The specifications are generally given in a _hardware description language_ (HDL). The two leading hardware description languages are _SystemVerilog_ and _VHDL_.

## Modules

A block of hardware with inputs and outputs is called a _module_. For example, an AND gate and a multiplexer, etc, are all hardware modules.

The two general styles for describing module functionality are _behavioral_ and _structural_.

* _Behavioral_ models describe what a module does.
* _Structural_ models decribe how a module is built from simpler pieces: it is an application of hierarchy.

Example 4.1 illustrate **behavioral** descriptions of a module that computes the Boolean function from $$y=\bar a\bar b\bar c+a\bar b\bar c+a\bar bc$$.

{% code title="Example 4.1 Combinational Logic" lineNumbers="true" %}
```verilog
module sillyfunction(input  logic a, b, c,
                     output logic y);
    assign y = ~a & ~b & ~c |
                a & ~b & ~c |
                a & ~b &  c;
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. A SystemVerilog module begins with the module name and a listing of inputs and outputs.
2. `logic` signals such as the inputs and outputs are Boolean vairables (0 or 1)
3. The `logic` type was introduced in SystemVerilog. It supersedes the `reg` type, which was a perennial source of confusion in Verilog. `logic` should be used everywhere except on signals [with multiple drivers](#user-content-fn-1)[^1]. Signals with multiple drivers are called `nets` and will explained later.
{% endhint %}

A module, as you might expect, is a good application of modularity. It has a well defined interface, consisting of its inputs and outputs, and it performs a specific function. The particular way in which it is coded is unimportant to others that might use the module, as long as it performs its function.

## Simulation and Synthesis

The two major purposes of HDLs are logic _simulation_ and _synthesis_.

* During **simulation**, inputs are applied to a module, and the outputs are checked to verify that the module operates correctly.
* During **synthesis**, the textual description of a module is transformed into logic gates.

### Simulation

Humans routinely make mistakes. Such errors in hardware designs are called _bugs_. These bugs are highly recommended to be eliminated before manufacturing the chips.

### Synthesis

Logic synthesis transforms HDL code into a _netlist_ describing the hardware (e.g., the logic gates and the wires connecting them). The netlist may be a text file, or it may be drawn as a schematic to help visualize the circuit.

***

In this book/course/note, our primary interest is to build hardware, we will emphasize a _synthesizable subset_ of the languages. Specifically, we will divide HDL code into _synthesizable_ modules and a _testbench_.

* The _synthesizable_ modules describe the hardware.
* The _testbench_ contains code to apply inputs to a module, check whether the output results are correct, and print discrepancies between expected and actual outputs. Testbench code is intended only for simulation and cannot be synthesized.

{% hint style="warning" %}
One of the most common mistakes for beginners is to think of HDL as a computer program rather than as a shorthand for describing digital hardware. So, think of your system in terms of blocks of combinational logic, registers, and finite state machines. Sketch these blocks on paper and show how they are connected before you start writing code.
{% endhint %}

HDLs have specific ways of describing various classes of logic; these ways are called _idioms_. This chapter will teach you how to write the proper HDL idioms for each type of block and then how to put the blocks together to produce a working system.

[^1]: If a signal is assigned by **two or more independent sources**, then it has **multiple drivers**. For example `assign x = a & b;`, `assign x = c & d;` , hence `x` has two drivers here and cannot be of type `logic`.
