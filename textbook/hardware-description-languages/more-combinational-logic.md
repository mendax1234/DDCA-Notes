# More Combinational Logic

`always` statements can also be used to describe combinational logic behaviorally if the sensitivity list is written to respond to changes in all of the inputs and the body prescribes the output value for every possible input combination.

HDLs support _blocking_ and _nonblocking assignments_ in an `always` statement.

* A group of blocking statements are evaluated in the order in which they appear in the code, just as one would expect in a standard programming language.
* A gourp of nonblocking assignments are evaluated **concurrently**; all of the statements are evaluated  before any of the signals on the left hand sides are updated.

{% hint style="info" %}
In a SystemVerilog `always` statement, `=` indicates a blocking assignment, and `<=` indicates a nonblocking assignment (also called a concurrent assignment).

Do not confuse either type with continuous assignment using the `assign` statement, `assign` statements must be used **outside** `always` statements and are also evaluated **concurrently**.
{% endhint %}

HDL Example 4.23 defines a full adder using intermediate signals `p` and `g` to compute `s` and `cout`. It produces the same circuit as code Example 4.7, but uses `always` statements in place of assignment statements.

{% code title="Example 4.23 Full Adder using always" lineNumbers="true" %}
```verilog
module fulladder(input  logic a, b, cin,
                 output logic s, cout);
  logic p, g;
  
  always_comb begin
    p    = a ^ b;        // blocking
    g    = a & b;        // blocking
    s    = p ^ cin;      // blocking
    cout = g | (p & cin); // blocking
  end
endmodule
```
{% endcode %}

{% hint style="info" %}
#### Code Explanation

1. For reasons that will be discussed later, in SystemVerilog `always` statement, it is best to use **blocking assignments** for combinational logic and **nonblocking assignments** for sequential logic.
{% endhint %}

## Case statements

A better application of using `always` statement for combinational logic is a seven-segment display that takes advantage of the `case` statement that **must appear inside** an `always` statement.

HDL Example 4.24 uses `case` statements to describe a seven-segment display decoder based on its truth table.

{% code title="Example 4.24 Seven-segment Display Decoder" lineNumbers="true" %}
```verilog
module sevenseg(input  logic [3:0] data,
                output logic [6:0] segments);
  always_comb
    case (data)
      // abc_defg
      0: segments = 7'b111_1110;
      1: segments = 7'b011_0000;
      2: segments = 7'b110_1101;
      3: segments = 7'b111_1001;
      4: segments = 7'b011_0011;
      5: segments = 7'b101_1011;
      6: segments = 7'b101_1111;
      7: segments = 7'b111_0000;
      8: segments = 7'b111_1111;
      9: segments = 7'b111_0011;
      default: segments = 7'b000_0000;
    endcase
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. The `default` clause is a convenient way to define the output for all cases not explicitly listed, guaranteeing combinational logic.
2. In SystemVerilog, `case` statement must appear inside `always` statements.
{% endhint %}

A `case` statement implies combinational logic if all possible input combinations are defined; otherwise it implies sequential logic, because the output will keep its old value in the undefined cases.

## If statements

`always` statements may also contain `if` statements. The `if` statement may be followed by an `else` statement. If all possible input combinations are handled, the statement implies combinational logic; otherwise, it produces sequential logic (like the [latch in the previous section](sequential-logic.md#latches))

{% hint style="warning" %}
In SystemVerilog, `if` statements must appear inside of `always` statements.
{% endhint %}

## Truth Tables with Don't Cares

As examined previously, truth tables may include don't care's to allow more logic simplification. HDL Example 4.27 shows how to describe a priority circuit with don't cares.

{% code title="Example 4.27 Priority Circuit using Don't Cares" lineNumbers="true" %}
```verilog
module priority_casez(input  logic [3:0] a,
                      output logic [3:0] y);
  always_comb
    casez (a)
      4'b1???: y <= 4'b1000;
      4'b01??: y <= 4'b0100;
      4'b001?: y <= 4'b0010;
      4'b0001: y <= 4'b0001;
      default: y <= 4'b0000;
    endcase
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. The `casez` statement acts like a `case` statement except that it also recognizes `?` as don't care.
{% endhint %}

## Blocking and Nonblocking Assignments

The following guidelines explain when and how to use each type of assignment. If these guidelines are not followed, it is possible to write code that appears to work in simulation but synthesizes to incorrect hardware.

{% stepper %}
{% step %}
**Use** `always_ff @(posedge clk)` **and nonblocking assigments to model synchronous sequential logic**

{% code lineNumbers="true" %}
```verilog
always_ff @(posedge clk) begin
  n1 <= d;  // nonblocking
  q  <= n1; // nonblocking
end
```
{% endcode %}
{% endstep %}

{% step %}
**Use continuous assignments to model simple combinational logic.**

```verilog
assign y = s ? d1 : d0;
```
{% endstep %}

{% step %}
**Use** `always_comb` **and blocking assignments to model more complicated combinational logic where the** `always` **statement is helpful.**

{% code lineNumbers="true" %}
```verilog
always_comb begin
  p    = a ^ b;        // blocking
  g    = a & b;        // blocking
  s    = p ^ cin;
  cout = g | (p & cin);
end
```
{% endcode %}
{% endstep %}

{% step %}
**Do not make assignments to the same signal in more than one** `always` **statement or continuous assignment statement.**
{% endstep %}
{% endstepper %}
