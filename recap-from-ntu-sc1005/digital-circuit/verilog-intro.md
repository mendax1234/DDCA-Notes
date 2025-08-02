# Verilog Intro

## Hardware Description Language

**H**ardware **D**escription **L**anguage, a.k.a HDL, is a **programming-like** language that is used to describe hardware.

HDLs are **synthesized** (and optimized) to hardware primitives while software languages are sometimes **compiled** to primitive instructions.

<details>

<summary>Verilog HDL vs. Software Program</summary>

As we have studied in CS1010, a software program, like a C program will be compiled into machine code. And this machine code, which is stored as an executable, can be executed directly by the CPU.

But for HDL, things are different.

</details>

## Verilog Basics

### Modules

* In Verilog, designs are broken down into **modules**
* A **module** is a **container** that the designer can use to **encapsulate** a unit of functionality.
* Modules can contain code to describe hardware and also instances of other modules.
  * Good designs consist of sufficient (but not excessive) **levels of hierarchy**, with modules containing instances of modules, that contain instances of other modules.
* At each level in the hierarchy, a module instance is treated as a "black-box" — the internals are unknown.

<figure><img src="../../.gitbook/assets/verilog-modules.jpg" alt=""><figcaption></figcaption></figure>

#### Verilog Module Declaration

{% stepper %}
{% step %}
**Use the** `module` **keyword and a list of ports**

```verilog
module somename (port1, port2, port3);
```

The above declaration describes a module with 3 **ports**, each a single wire
{% endstep %}

{% step %}
**Describe the direction of ports using** `input` **and** `output`

```verilog
module somename (input port1, port2,
                 output port3);
```

The above describes a module with two inputs and one output.
{% endstep %}

{% step %}
`endmodule` **keyword**

The `endmodule` keyword indicates the end of statements that comprise the module description.
{% endstep %}
{% endstepper %}

#### Gate-Level Primitives

Verilog provides us with basic **primitives** to model Boolean gates: `and`, `nand`, `or`, `nor`, `not`, `xor`, `xnor`.

{% stepper %}
{% step %}
**Gate-Level Primitives Example**

The following Verilog code represents an `and` gate with inputs connected to wires `a` and `b`, and output connected to wire `y`.

<figure><img src="../../.gitbook/assets/gate-level-primitives-example.jpg" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
The [**single**](#user-content-fn-1)[^1] output is always the **first argument.**
{% endhint %}
{% endstep %}

{% step %}
**Wires**

We can declare **internal wires** in a module, using the `wire` keyword

```verilog
wire int_signal;
```

This creates a named (1-bit) wire that can be used to connect gates together.
{% endstep %}

{% step %}
**Examples with** `wire`

To describe the following circuit,

<figure><img src="../../.gitbook/assets/wire-example.jpg" alt="" width="563"><figcaption></figcaption></figure>

The HDL we write will be

```verilog
module andorinv (input a, b, c, d,
                 output out);
       wire and1out, and2out;
       
       and (and1out, a, b);
       and (and2out, c, d);
       nor (out, and1out, and2out);
endmodule
```

{% hint style="info" %}
Note that we can also add **gate identifiers** for each gate. And in the following code, `g1`, `g2`, and `g3` are three gate identifiers.

```verilog
module andorinv (input a, b, c, d,
                 output out);
       wire and1out, and2out;
       
       and g1 (and1out, a, b);
       and g2 (and2out, c, d);
       nor g3 (out, and1out, and2out);
endmodule
```
{% endhint %}
{% endstep %}

{% step %}
**Notes for this part**

1. The process of using Verilog HDL to describe the gate-level circuit directly is called **structural Verilog**.
2. In the Verilog `module`, the **order** of statements is **unimportant**. Each statement describes a piece of hardware. There is **no sequence of steps** when doing **structural Verilog**.&#x20;
3.  In the Verilog `wire`, usually there is no need to declare 1-bit wires for connecting modules. Tools will infer these wires automatically by matching names.

    1. This only applies for 1-bit wires in instantiation.

    <figure><img src="../../.gitbook/assets/sequence-dn-matter-in-verilog.jpg" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

[^1]: Why single? Because a gate cannot have two outputs.
