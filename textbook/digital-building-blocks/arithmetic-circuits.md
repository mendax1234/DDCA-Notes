# Arithmetic Circuits

Arithmetic circuits are the central building blocks of computers. Computers and digital logic perform many arithmetic functions: addition, subtraction, comparisons, shifts, multiplication, and division. This section describes hardware implementations for all of these operations.

## Addition

Addition is one of the most common operations in digital systems. If you still remember, we've introduced [here](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#recap) that all binary arithmetics can be done using adders! We first consider how to add two 1-bit binary numbers. We then extend to N-bit binary numbers. Adders also illustrate trade-offs between speed and complexity.

### Half Adder&#x20;

We begin by building a 1-bit _half adder_. As shown in Figrue 5.1, the half adder has two inputs, A and B, and two outputs, $$S$$ and $$C_{\text{out}}$$. $$S$$ is the sum of A and B. If A and B is 1, $$S$$ is 2, which cannot be represented with a single binary digit. Instead, it is indicated with a carry out $$C_{\text{out}}$$ **in the next column**.

<figure><img src="../../.gitbook/assets/1-bit-half-adder.png" alt="" width="228"><figcaption></figcaption></figure>

In a multi-bit adder, $$C_{\text{out}}$$ is added or _carried in_ to the next most significant bit. (or into the next column) However, the half adder lacks a $$C_{\text{in}}$$ input to accept $$C_{\text{out}}$$ of the previous column. The _full adder_, described in the next section, solves this problem.

### Full Adder

A _full adder_ accepts the carry in $$C_{\text{in}}$$ as shown in Figure 5.3. The figure also shows the output equations for $$S$$ and $$C_{\text{out}}$$.

<figure><img src="../../.gitbook/assets/1-bit-full-adder.png" alt="" width="304"><figcaption></figcaption></figure>

### Carry Propagate Adder

An N-bit adders sums two N-bit inputs, A and B, and a carry in $$C_{\text{in}}$$ to produce an N-bit result $$S$$ and a carry out $$C_{\text{out}}$$. It is commonly called a _carry propagate adder_ (CPA) because the carry out of one bit propagates into the next bit. The symbol for CPA is shown in Figure 5.4; it is drawn just like a full adder except that A, B, and S are busses rather than single bits. Three common CPA implementations are called

1. ripple-carry adders
2. carry-lookahead adders
3. prefix adders (FYI)

<figure><img src="../../.gitbook/assets/carry-propagate-adder.png" alt=""><figcaption></figcaption></figure>

#### Ripple-Carry Adder

The simplest way to build an N-bit carry propagate adder is to chain together N full adders. The $$C_{\text{out}}$$ of one stage acts as the $$C_{\text{in}}$$ of the next stage, as shown in Figure 5.5 for 32-bit addition. This is called a _ripple-carry adder_.

<figure><img src="../../.gitbook/assets/32-bit-ripple-carry-adder.png" alt=""><figcaption></figcaption></figure>

The ripple-carry adder has the disadvantage of being slow when N is large. As $$S_{31}$$ depends on $$C_{30}$$, which depends on $$C_{29}$$, and so forth all the way back to $$C_{\text{in}}$$. We say that the carry _ripples_ through the carry chain. The delay of the adder, $$t_{\text{ripple}}$$ grows directly with the number of bits, as given in Equation 5.1, where $$t_{\text{FA}}$$ is the delay of a full adder,

$$
t_{\text{ripple}}=Nt_{\text{FA}}
$$

#### Carry-Lookahead Adder

A _carry-lookahead_ adder (CLA) is another type of carry propogate adder that solves the problem caused by ripple-carry adder by dividing the adder into _blocks_ and providing circuitry to quickly determine the carry out of a block as soon as the carry in is known. Thus it is said to _look ahead_ across the blocks rather than waiting to ripple through all the full adders inside a block.

CLAs use _generate_ (G) and _propogate_ (P) signals that describe how a column or block determines the carry out.

* The ith column of an adder is said to _generate_ a carry if it produces a carry out independent of the carry in. Hence, we have $$G_i=A_iB_i$$.
* The ith column of an adder is said to _propagate_ a carry it is produces a carry [whenever there is a carry in](#user-content-fn-1)[^1]. Thus, we have $$P_i=A_i+B_i$$.

Figure 5.6 (a) shows a 32-bit carry-lookahead adder composed of eight 4-bit blocks. Each block contains a 4-bit ripple-carry adder and some lookahead logic to compute the carry out of the block given the carry in, as shown in Figure 5.6 (b). The AND and OR gates needed to compute the single-bit generate and propagate signals, $$G_i$$ and $$P_i$$, from $$A_i$$ and $$B_i$$ are left out for brevity.

<figure><img src="../../.gitbook/assets/32-bit-carry-lookahead-adder.png" alt=""><figcaption></figcaption></figure>

### Putting it all together

HDLs provide the `+` operation to specify a CPA. HDL Example 5.1 describes a CPA with carries in and out

{% code title="Example 5.1 Adder" lineNumbers="true" %}
```verilog
module adder #(parameter N = 8)
  (input  logic [N-1:0] a, b,
   input  logic         cin,
   output logic [N-1:0] s,
   output logic         cout);
  assign {cout, s} = a + b + cin;
endmodule
```
{% endcode %}

## Subtraction

Recall from [previous part](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#signed-binary-number-representation) that adders can add positive and negative numbers using [two's complement number representation](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#id-2s-complement). Substraction is almost as easy:

1. flip the sign of teh second number
2. then add

{% hint style="info" %}
Flipping the sign of a two's complement number is done by inverting the bits and adding 1.
{% endhint %}

Thus, subtraction is performed with a single CPA by adding $$A+\bar B$$ with $$C_{\text{in}}=1$$. Figure 5.9 shows the symbol for a subtractor and the underlying hardware for performing $$Y=A-B$$. HDL Example 5.2 describes a subtractor.

<figure><img src="../../.gitbook/assets/n-bit-subtractor.png" alt=""><figcaption></figcaption></figure>

{% code title="Example 5.2 Subtractor" lineNumbers="true" %}
```verilog
module subtractor #(parameter N = 8)
                   (input  logic [N-1:0] a, b,
                    output logic [N-1:0] y);
  assign y = a - b;
endmodule
```
{% endcode %}

## Comparators

A _comparator_ determines whether two binary numbers are equal or if one is greater or less than the other. A comparator receives two N-bit binary numbers A and B. There are two common types of comparators.

* A _equality comparator_ produces a single output indicating whether A is equal to B (`A==B`).
* A _magnitude comparator_ produces one or more outpus indicating the relative values of A and B.

{% stepper %}
{% step %}
**Equality comparator**

The eqaulity comparator is the simpler piece of hardware. Figure 5.11 shows the symbol and implementation of a 4-bit equality comparator. It first checks to determine whether the corresponding btis in each column of A and B are equal using XNOR gates. The numbers are equal if all of the columns are equal.

<figure><img src="../../.gitbook/assets/equality-comparator.png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**Magnitude comparator**

Magnitude comparison is usually done by computing `A-B` and looking at the sign (most significant bit) of the resulf as shown in Figure 5.12. If the result is negative (e.g., the sign bit is 1), then A is less than B. Otherwise A is greater than or equal to B.

<figure><img src="../../.gitbook/assets/magnitude-comparator.png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

HDL Example 5.3 shows how to ues various comparison operations.

{% code title="Example 5.3 Comparators" lineNumbers="true" %}
```verilog
module comparator #(parameter N = 8)
                   (input  logic [N-1:0] a, b,
                    output logic        eq, neq, lt, lte, gt, gte);
  assign eq  = (a == b);
  assign neq = (a != b);
  assign lt  = (a < b);
  assign lte = (a <= b);
  assign gt  = (a > b);
  assign gte = (a >= b);
endmodule
```
{% endcode %}

## ALU

An _Arithmetic/Logical Unit_ (ALU) combines a variety of mathematical and logical operations into a single unit. For example, a typical ALU might perform addition, subtraction, magnitude comparison, AND, and OR operations. The ALU forms the heart of most computer systems.

Figure 5.14 shows the symbol for an N-bit ALU with N-bit inputs and outputs.

<figure><img src="../../.gitbook/assets/alu-symbol.png" alt=""><figcaption></figcaption></figure>

The ALU receives a control signal F that specifies which function to perform. Table 5.1 lists typical functions that the ALU can perform.

<figure><img src="../../.gitbook/assets/alu-operations.png" alt="" width="394"><figcaption></figcaption></figure>

Figure 5.15 shows an implementation of the ALU.

<figure><img src="../../.gitbook/assets/n-bit-alu.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
#### Notes

1. $$F_2$$ is also the carry in to the adder.
2. In two's complement arithmetic, $$\bar B+1=-B$$.
3. When $$F_{\text{[2:0]}}=111$$, the ALU performs the _set if less than_ (SLT) operation. When $$A<B$$, $$Y=1$$. Otherwise, $$Y=0$$. In other words, Y is set to 1 if A is less than B.
   1. SLT is performed by computing `S=A-B`. If S is negative (e.g., the sign bit is set), A is less than B. The _zero extend unit_ produces an N-bit output by concatenating is 1-bit input with 0's in the most significant bits. The sign bit ( $$(N-1)^{\text{th}}$$ bit) of S is the input to the zero extend unit.
{% endhint %}

Some ALUs produce extra outputs, called _flags_, that indicate information about the ALU output. For example, an _overflow flag_ indicates that the result of the adder overflowed. A _zero flag_ indicates that teh ALU output is 0.

The HDL for an N-bit ALU is shown as follows

{% code lineNumbers="true" %}
```verilog
module alu32(input  logic [31:0] A, B,
             input  logic [2:0]  F,
             output logic [31:0] Y);
  logic [31:0] S, Bout;
  
  assign Bout = F[2] ? ~B : B;
  assign S = A + Bout + F[2];
  
  always_comb
    case (F[1:0])
      2'b00: Y <= A & Bout;
      2'b01: Y <= A | Bout;
      2'b10: Y <= S;
      2'b11: Y <= S[31];
    endcase
endmodule
```
{% endcode %}

## Shifters and Rotators

_Shifters_ and _rotators_ move bits and multiply or divide by powers of 2. There are several kinds of commonly used shifters:

{% stepper %}
{% step %}
**Logical shifter**

It shifts the number to the left (LSL) or right (LSR) and fills empty spots with 0's.

Ex. 11001 LSR 2 = 00110; 11001 LSL 2 = 00100;
{% endstep %}

{% step %}
**Arithmetic shifter**

It is the same as logical shifter, but on right shifts fills the most significant bits with a copy of the old most significant bit (msb). This is useful for multiplying and dividing signed numbers. Arithmetic shift left (ASL) is the same as logical shift left (LSL) because we will fill the empty spots at right with 0, this doesn't matter much.

Ex. 11001 ASR 2 = 11110; 11001 ASL 2 = 00100;
{% endstep %}

{% step %}
**Rotator**

It rotates number in circle such that empty spots are filled with bits shifted off the other end.

Ex. 11001 ROR 2 = 01110; 11001 ROL 2 = 00111;
{% endstep %}
{% endstepper %}

A N-bit shifter can be built from N N:1 multiplexers. This input is shifted by 0 to N-1 bits, depending on the value of the $$\log_2N$$-bit select lines. Figure 5.16 shows the symbol and hardware of 4-bit shifters. The operators `<<`, `>>`, and `>>>` typically indicate shift left, logical shift right, and arithmetic shift right, respectively.

<figure><img src="../../.gitbook/assets/4-bit-shifters.png" alt=""><figcaption></figcaption></figure>

## Multiplication

In the [previous part](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#multiplication), we have seen that multiplication is nothing but shift then add, by shift, we form the _partial products_.

In general, a NxN multiplier multiplies two N-bit numbers and produces a 2N-bit result. Multiplication of 1-bit binary numbers is equivalent to the AND operation, so AND gates are used to form the partial products.

Figure 5.18 shows the symbol, function, and implementation of a 4x4 multiplier.

<figure><img src="../../.gitbook/assets/4x4-multiplier.png" alt=""><figcaption></figcaption></figure>

The HDL for a multiplier is in HDL Example 5.4. As with adders, the synthesis tools may pick the most appropriate design given the timing constraints.

{% code title="Example 5.4 Multiplie" lineNumbers="true" %}
```verilog
module multiplier #(parameter N = 8)
                   (input  logic [N-1:0]    a, b,
                    output logic [2*N-1:0] y);
  assign y = a * b;
endmodule
```
{% endcode %}

## Division

FYI part first.

[^1]: This means, when there is a carry in, the column will always produce a carry out, which **propagates** the carry in it receives.
