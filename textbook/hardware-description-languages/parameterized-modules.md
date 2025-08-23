# Parameterized Modules

So far all of our modules have had fixed-width inputs and outputs. For example, we had to define separate modules for 4- and 8-bit wide 2:1 multiplexers. HDLs permit variable bit widths using parameterized modules.

HDL Example 4.34 declares a parameterized 2:1 multiplexer with a default width of 8, then uses it to create 12-bit 4:1 multiplexers.

{% code title="Example 4.34 Parameterized N-Bit 2:1 Multiplexer" lineNumbers="true" %}
```verilog
module mux2
  #(parameter width = 8)
  (input  logic [width-1:0] d0, d1,
   input  logic             s,
   output logic [width-1:0] y);
  assign y = s ? d1 : d0;
endmodule

module mux4_12(input  logic [11:0] d0, d1, d2, d3,
               input  logic [1:0]  s,
               output logic [11:0] y);
  logic [11:0] low, hi;
  
  mux2 #(12) lowmux(d0, d1, s[0], low);
  mux2 #(12) himux(d2, d3, s[0], hi);
  mux2 #(12) outmux(low, hi, s[1], y);
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. A 12-bit 4:1 multiplexer, `mux4_12`, would need to override the default width using `#()` before the instance name.
{% endhint %}
