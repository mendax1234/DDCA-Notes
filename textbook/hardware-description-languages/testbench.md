# Testbench

As we have seen [before](https://wenbo-notes.gitbook.io/ddca-notes/textbook/hardware-description-languages#synthesis), HDL will code be divided into synthesizable and testbench. A _testbench_ is an HDL module that is used to test another module, called the _device under test (DUT)_.

{% hint style="info" %}
Some tools also call the module to be tested the _unit under test (UUT)_.
{% endhint %}

HDL example 4.38 shows how to write a self-checking testbench,

{% code title="Example 4.38 Self-Checking Testbench" lineNumbers="true" %}
```verilog
module testbench2();
  logic a, b, c, y;
  
  // instantiate device under test
  sillyfunction dut(a, b, c, y);
  
  // apply inputs one at a time
  // checking results
  initial begin
    a = 0; b = 0; c = 0; #10;
    assert (y === 1) else $error("000 failed.");
    
    c = 1; #10;
    assert (y === 0) else $error("001 failed.");
    
    b = 1; c = 0; #10;
    assert (y === 0) else $error("010 failed.");
    
    c = 1; #10;
    assert (y === 0) else $error("011 failed.");
    
    a = 1; b = 0; c = 0; #10;
    assert (y === 1) else $error("100 failed.");
    
    c = 1; #10;
    assert (y === 1) else $error("101 failed.");
    
    b = 1; c = 0; #10;
    assert (y === 0) else $error("110 failed.");
    
    c = 1; #10;
    assert (y === 0) else $error("111 failed.");
  end
endmodule
```
{% endcode %}

{% hint style="info" %}
#### Code Explanation

1. The `initial` statement executes the statements in its body at the start of simulation.
2. `initial` statement should be used only in testbenches for simulation, not in modules intended to be synthesized into actual hardware.
3. The blocking statements (`=`) and delays (`#`) are used to apply the inputs in the appropriate order.
4. Testbenches uses `===` and `!==` (instead of `==` and `!=`) for comparisons of equality and inequality, respectively, because these operators work correctly with operands that could be `x` or `z`.
{% endhint %}
