# Data Types

In Verilog (not SystemVerilog), there are two types: `reg` and `wire`. This was a great source of confusion for those learning the language (including me😂). This section will explain `reg` and `wire`.

## `reg` and `wire` Rule of Thumb

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

## Wire

Unlike physical wires, wires (and other signals) in Verilog are _directional_. This means information flows in only one direction, from (usually one) _source_ to the _sinks_ (The source is also often called a _driver_ that _drives_ a value onto a wire).

### Vector

Vectors are used to group related signals using one name to make it more convenient to manipulate. For example, `wire [7:0] w;` declares an 8-bit vector named `w` that is functionally equivalent to having 8 separate wires.

{% hint style="info" %}
In Verilog, vector is just a group of wires.
{% endhint %}

For example, for the circuit as follows, its verilog code will be

<figure><img src="../../.gitbook/assets/verilog-vector-example.png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```verilog
module top_module(
	input [2:0] vec, 
	output [2:0] outv,
	output o2,
	output o1,
	output o0
);
	
	assign outv = vec;

	// This is ok too: assign {o2, o1, o0} = vec;
	assign o0 = vec[0];
	assign o1 = vec[1];
	assign o2 = vec[2];
	
endmodule
```
{% endcode %}
