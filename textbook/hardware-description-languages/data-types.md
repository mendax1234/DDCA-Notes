# Data Types

In Verilog (not SystemVerilog), there are two types: `reg` and `wire`. This was a great source of confusion for those learning the language (including me😂). This section will explain `reg` and `wire`.

## `reg` and `wire`

The rule of thumb is,

> In Verilog, if a signal appears on the left hand side of `<=` or `=` in an `always` block, it must be declared as `reg`. Otherwise, it should be declared as `wire`.

Hence, a `reg` signal might be the output of a flip-flop, a latch, or combinational logic, depending on the sensitivity list and statement of an `always` block.

> Input and output ports default to the `wire` type unless their type is explicitly defined as `reg`.

For example, the following code shows how a flip-flop is defined in Verilog.

{% code lineNumbers="true" %}
```verilog
module flop(input            clk,
            input      [3:0] d,
            output reg [3:0] q);
  always @(posedge clk)
    q <= d;
endmodule
```
{% endcode %}

***

Verilog/SystemVerilog has much more stuff other than the things we've covered so far, for your interest, can jump to [more-verilog.md](more-verilog.md "mention")!
