# FPGA Design Tips

## RTL Coding Steps

### Theoretical

This part has been introduced in [Lec 02](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-02-digital-system-design-and-verilog#behavioral-modelling-1)! Why theoretical, I guess it's because it is introduced during the lec. 😂 But that doesn't mean it's not important, it's still very important!

### Practical

We have introduced the FPGA Design Flow in [CS2100DE Lab 01](https://wenbo-notes.gitbook.io/ddca-notes/lab/preparation-cs2100de/lab-01#the-fpga-design-flow). Here, I want to add more steps when we write our RTL Code.

{% stepper %}
{% step %}
**Know your design specification**

Before we write anything, clarify our **design specification**, which is mainly about

1. what **inputs** we will use -> how we will control our FPGA board
   1. What buttons will be used? (`btnR`, `btnC`, etc)
   2. What switches will be used? (`sw[2:0]`, etc)
2. what **outputs** we will use -> how we want to display on our FPGA board
   1. Is it the seven-segment display? (`an[7:0]`, `seg[6:0]`)
   2. Is it the LED? (`led[2:0]`, etc)

These are the very important things we need to clarify before actually writing our FPGA code.

{% hint style="success" %}
These inputs and outputs are then specified in our `Top.sv` or `Top.v` module.
{% endhint %}
{% endstep %}

{% step %}
**Design the inner logic**

In Verilog, the simplest buliding is called [_modules_](https://wenbo-notes.gitbook.io/ddca-notes/textbook/hardware-description-languages#modules). Our top module is `Top.sv` / `Top.v`, and it is composed of many other instantiated small modules, this is called [_structural_ modeling](https://wenbo-notes.gitbook.io/ddca-notes/textbook/hardware-description-languages#modules). In this step, we mainly care about

1. How each small module interact with others and our top module's inputs/outputs.

Define new variables or new combinational/sequential logic if it is necessary.

{% hint style="success" %}
This step should also be done in our top module `Top.sv` or `Top.v`.
{% endhint %}
{% endstep %}

{% step %}
**Make the smaller module as flexible as possible**

In structural modeling, the smaller modules are used very often. Thus, it will be a good idea to make them as flexible as they can, by [parameterizing the modules](https://wenbo-notes.gitbook.io/ddca-notes/textbook/hardware-description-languages/parameterized-modules).

{% hint style="success" %}
A more concrete example about parameterized module is [here](https://wenbo-notes.gitbook.io/ddca-notes/lab/preparation-cs2100de/lab-03#customized-speed-counter).
{% endhint %}
{% endstep %}
{% endstepper %}
