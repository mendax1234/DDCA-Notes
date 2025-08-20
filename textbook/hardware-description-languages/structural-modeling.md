# Structural Modeling

The previous section discussed _behavioral_ modeling, describing a module in terms of the relationships between inputs and outputs. This section examines _structural_ modeling, describing a module in terms of how it is composed of simpler modules.

For example, HDL Example 4.14 shows how to assemble a 4:1 multiplexer from three 2:1 multiplexers. Each copy of 2:1 multiplexer is called an _instance_. Multiple instances of the same module are distinguished by distinct names, in this case `lowmux`, `highmux`, and `finalmux`. This is an example of regularity, in which the 2:1 multiplexer is reused many times.

{% code title="Example 4.14 Structural Model of 4:1 Multiplexer" lineNumbers="true" %}
```verilog
module mux4(input  logic [3:0] d0, d1, d2, d3,
            input  logic [1:0] s,
            output logic [3:0] y);
  logic [3:0] low, high;
  
  mux2 lowmux(d0, d1, s[0], low);
  mux2 highmux(d2, d3, s[0], high);
  mux2 finalmux(low, high, s[1], y);
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. The `mux2` module must be defined elsewhere in the SystemVerilog code — see HDL Example 4.5.
{% endhint %}

In general, complex systems are designed _hierarchically_. The overall system is described structurally by instantiating its major components. Each of these components is described structurally from its building  blocks, and so forth recursively until the pieces are simple enough to describe behaviorally.

{% hint style="info" %}
It is a good style to avoid (or at least to minimize) mixing structural and behavioral descriptions within a single module.
{% endhint %}
