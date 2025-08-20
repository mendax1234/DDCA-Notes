# Sequential Logic

HDL synthesizers recognize certain idioms and turn them into specific sequential circuits. Other coding styles may simulate correctly but synthesize into circuits with blatant or subtle errors. This section presents the proper idioms to describe registers and latches.

## Registers

The vast majority of modern commercial systems are built with registers using positive edge-triggered D flip-flops. HDL Example 4.17 shows the idiom for such flip-flops.

In SystemVerilog `always` statements, signals keep their old value until an event in the sensitivity list takes place that explicitly causes them to change. Hence, such code, with appropriate sensitivity lists, can be used to describe sequential circuits with memory. For example, the flip-flop includes only `clk` in the sensitive list. It remembers its old value  of `q` until the next rising edge of the `clk`, even if `d` changes in the interim.

In contrast, SystemVerilog continuous assignment statements (`assign`) are reevaluated **anytime** **any of the inputs** on the right hand side changes. Therefore, such code necessarily describes combinational logic.

{% code title="Example 4.17 Register" lineNumbers="true" %}
```verilog
module flop(input  logic       clk,
            input  logic [3:0] d,
            output logic [3:0] q);
  always_ff @(posedge clk)
    q <= d;
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1.  In general, a SystemVerilog `always` statement is written in the form\


    {% code lineNumbers="true" %}
    ```verilog
    always @(sensitivity list)
      statement;
    ```
    {% endcode %}
2. The `statement` is executed **only when** the event specified in the `sensitivity list` occurs. In this example, the statement is `q <= d` (pronounced "q gets d"). Hence, the flip-flop copies `d` to `q` on the positive edge of the clock and otherwise remembers the old state of `q`. Note that sensitivity lists are also referred to as stimulus lists.
3. `<=` is called a _nonblocking assignment_. Think of it as a regular `=` sign for now; we'll return to the more subtle points later. Note that `<=` is used instead of `assign` inside an `always` statement.
4. As will be seen in subsequent sections, `always` statements can be used to imply flip-flops, latches, or combinational logic, depending on the **sensitivity list** and **statement**. Because of this flexibility, it is easy to produce the wrong hardware inadvertently. SystemVerilog introduces `always_ff`, `always_latch`, and `always_comb` to reduce the risk of common errors. `always_ff` behaves like `always` but is used exclusively to imply flip-flops and allows tools to produce a warning if anything else is implied.
{% endhint %}

### Resettable Registers

When simulation begins or power is first applied to a circuit, the output of a flop or register is unknown. This is indicated with `x` in SystemVerilog. Generally, it is a good practice to use resettable registers so that on powerup you can put your system in a known state. The reset may be either **asynchronous** or **synchronous**. Recall that asynchronous reset occurs immediately, wheras synchronous reset clears the output only on the next rising edge of the clock. HDL Example 4.18 demonstrates the idioms for flip-flops with asynchronous and synchronous resets.

{% code title="Example 4.18 Resettable Register" lineNumbers="true" %}
```verilog
module flopr(input  logic       clk,
             input  logic       reset,
             input  logic [3:0] d,
             output logic [3:0] q);
  // asynchronous reset
  always_ff @(posedge clk, posedge reset)
    if (reset) q <= 4'b0;
    else       q <= d;
endmodule

module flopr(input  logic       clk,
             input  logic       reset,
             input  logic [3:0] d,
             output logic [3:0] q);
  // synchronous reset
  always_ff @(posedge clk)
    if (reset) q <= 4'b0;
    else       q <= d;
endmodule
```
{% endcode %}

### Enabled Registers

Enabled registers respond to the clock only when the enable is asserted. HDL Example 4.19 shows an asynchronously resettable enabled register that retains its old value if both `reset` and `en` are FALSE.

{% code title="Example 4.19 Resettable Enabled Register" lineNumbers="true" %}
```verilog
module flopenr(input  logic       clk,
               input  logic       reset,
               input  logic       en,
               input  logic [3:0] d,
               output logic [3:0] q);
  // asynchronous reset
  always_ff @(posedge clk, posedge reset)
    if (reset)   q <= 4'b0;
    else if (en) q <= d;
endmodule
```
{% endcode %}

### Multiple Registers

A single `always` statement in SystemVerilog can be used to describe multiple pieces of hardware. For example, consider the synchronizer amde of two back-to-back flip-flops, as shown in Figure 4.17.

<figure><img src="../../.gitbook/assets/synchronizer.png" alt=""><figcaption></figcaption></figure>

HDL Example 4.20 describes the synchronizer. On the rising edge of `clk`, `d` is copied to `n1`. At the same time, `n1` is copied to `q`.

{% code title="Example 4.20 Synchronizer" lineNumbers="true" %}
```verilog
module sync(input  logic clk,
            input  logic d,
            output logic q);
  logic n1;
  
  always_ff @(posedge clk) begin
    n1 <= d;  // nonblocking
    q  <= n1; // nonblocking
  end
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. Notice that the `begin/end` construct is **necessary** because multiple statements appear in the `always` statement. This is analogous to `{}` in C or Java. The `begin/end` was not needed in the `flopr` example because `if/else` counts as a single statement.
{% endhint %}

## Latches

Recall from [previous section](../sequential-logic-design/latches-and-flip-flops.md#d-latch) that a D latch is transparent when the clock is HIGH, allowing data to flow from input to output. The latch becomes opaque when the clock is LOW, retaining its old state. HDL Example 4.21 shows the idiom of a D latch.&#x20;

{% code title="Example 4.21 D Latch" lineNumbers="true" %}
```verilog
// Some code
```
{% endcode %}

{% hint style="info" %}
#### Code Explanation

1. `always_latch` is equivalent to `always @(clk, d)` and is the preferred idiom for describing a latch in SystemVerilog. It evaluates any time `clk` or `d` changes. If `clk` is HIGH, `d` flows through to `q`, so this code describes a positive level sensitive latch. Otherwise, `q` keeps its old value.
{% endhint %}

Not all synthesis tools supports latches well. Unless you know that your tool does support latches and you have a good reason to use the, **avoid them and use edge-triggered flip-flops instead**.
