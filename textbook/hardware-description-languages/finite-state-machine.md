# Finite State Machine

Recall that a FSM contains three main parts:

1. a state register
2. **two** blocks of combinational logic

HDL Example 4.30 describes the divide-by-3 FSM.

{% tabs %}
{% tab title="SystemVerilog" %}
{% code title="Example 4.30 Divide-by-3 Finite State Machine" lineNumbers="true" %}
```verilog
module divideby3FSM(input  logic clk,
                    input  logic reset,
                    output logic y);
  typedef enum logic [1:0] {S0, S1, S2} statetype;
  statetype [1:0] state, nextstate;
  
  // state register
  always_ff @(posedge clk, posedge reset)
    if (reset) state <= S0;
    else       state <= nextstate;
  
  // next state logic
  always_comb
    case (state)
      S0:       nextstate <= S1;
      S1:       nextstate <= S2;
      S2:       nextstate <= S0;
      default:  nextstate <= S0;
    endcase
  
  // output logic
  assign y = (state == S0);
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. **(SystemVerilog Specific)** The `typedef` statement defines `statetype` to be a two-bit `logic` value with three possibilities: `S0`, `S1`, or `S2`. `state` and `nextstate` are `statetype` signals.
2. **(SystemVerilog Specific)** The enumerated encodings default to numerical order: `S0=00`, `S1=01`, and `S2=10`.
3. Notice how a `case` statement is used to define the state transition table. Because the next state logic should be combinational, a `default` is necessary even though the state `2'b11` should never arise.
{% endhint %}
{% endtab %}

{% tab title="Verilog" %}
{% code title="Example 4.30 Divide-by-3 Finite State Machine" lineNumbers="true" %}
```verilog
module divideby3FSM(input  clk,
                    input  reset,
                    output y);
  reg [1:0] state, nextstate;
  parameter S0 = 2'b00;
  parameter S1 = 2'b01;
  parameter S2 = 2'b10;
  
  // state register
  always @(posedge clk, posedge reset)
    if (reset) state <= S0;
    else       state <= nextstate;
  
  // next state logic
  always @(*)
    case (state)
      S0:       nextstate = S1;
      S1:       nextstate = S2;
      S2:       nextstate = S0;
      default:  nextstate = S0;
    endcase
  
  // output logic
  assign y = (state == S0);
endmodule
```
{% endcode %}

{% hint style="success" %}
#### Code Explanation

1. The `parameter` statement is used to define constants within a module. Naming the states with parameters is not required, but it makes changing state encodings much easier and makes the code more readable.
{% endhint %}
{% endtab %}
{% endtabs %}
