# More Verilog

As we have a glimpse of how SystemVerilog/Verilog looks like by looking at how they are constructed by using the digital building blocks, like combinational logic, registers, etc. Now, we shall also introduce the programming side of this Verilog.

> Most content are from [HDLBits](https://hdlbits.01xz.net/wiki/Main_Page)!

## Wire

Unlike physical wires, wires (and other signals) in Verilog are _directional_. This means information flows in only one direction, from (usually one) _source_ to the _sinks_ (The source is also often called a _driver_ that _drives_ a value onto a wire).

The ports on a module also have a direction (usually input or output). An input port is _driven by_ something from outside the module, while an output port _drives_ something outside. When viewed from inside the module, an input port is a driver or source, while an output port is a sink.

<figure><img src="../../.gitbook/assets/verilog-wire.png" alt=""><figcaption></figcaption></figure>

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
