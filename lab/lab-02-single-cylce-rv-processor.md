# Lab 02 - Single Cylce RV Processor

## Introduction

In this lab, we will build a single-cycle RISC-V processor and the lab manual is [here](https://nus-cg3207.github.io/labs/asst_manuals/Asst_02/Asst_02/). And our processor will support the followig instructions for now,

* `add`, `addi`, `sub`, `and`, `andi`, `or`, `ori`
* `lw`, `sw`
* `beq`, `bne`, `jal` (without linking, that is, without saving the return address).&#x20;
* `lui`,`auipc`
* `sll`, `srl`, `sra`

## Tips

### Demo

As you may be confused about what you are going to demo, here are some tips from the teaching team

{% stepper %}
{% step %}
#### General Caveats

1. Make your output **dependant on** some user inputs. A.k.a, don't hardcode the output using a counter, etc.
2. Design your assembly program such that if one instruction doesn't work, the final output will be completely different. In other words, to get this output correctly given this input, every instruction in your assembly program should work correctly.
   1. **Don't just look at the final output** to state that every instruction of your prgram works. For example, if you shift right by 5 bits and shift left by 5 bits, even if your shift instructions don't work, your result will still be right.
{% endstep %}

{% step %}
#### Testbench

Your progam should run based on your user input, so the testbench just serves as **simulating** that user input and check whether the output of your program is correct or not.
{% endstep %}

{% step %}
#### More about OLED

In the lab, OLED is provided, but it is just another output. The fancy demo you may see from the lab by Dr. Rajesh is just another way to craft the **well-designed** assembly code.
{% endstep %}

{% step %}
#### General Steps

{% hint style="success" %}
This is the most important tip for demo!
{% endhint %}

1. Assembly code correctness: Step through the assembly program step by step to make sure the algorithm works correctly on the "high-level".
2. HDL (Vivado) simulation: Step through the each instruction to make sure each instruction is executed correctly on vivado simulation (behavioral + post-synthesis)
3. Hardware demo: This step **cannot step through** and thus we just provide the input and see the output.
{% endstep %}
{% endstepper %}
