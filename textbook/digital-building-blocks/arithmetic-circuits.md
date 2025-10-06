# Arithmetic Circuits

> This part is almost the same NUS CG3207 Lec 04 - Arithmetic for Computers. So, I will combine the information together here.

Arithmetic circuits are the central building blocks of computers. Computers and digital logic perform many arithmetic functions: addition, subtraction, comparisons, shifts, multiplication, and division. This section describes hardware implementations for all of these operations.

## Addition

Addition is one of the most common operations in digital systems. If you still remember, we've introduced [here](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#recap) that all binary arithmetics can be done using adders! We first consider how to add two 1-bit binary numbers. We then extend to N-bit binary numbers. Adders also illustrate trade-offs between speed and complexity.

### Half Adder&#x20;

We begin by building a 1-bit _half adder_. As shown in Figrue 5.1, the half adder has two inputs, A and B, and two outputs, $$S$$ and $$C_{\text{out}}$$. $$S$$ is the sum of A and B. If A and B is 1, $$S$$ is 2, which cannot be represented with a single binary digit. Instead, it is indicated with a carry out $$C_{\text{out}}$$ **in the next column**.

<figure><img src="../../.gitbook/assets/1-bit-half-adder.png" alt="" width="228"><figcaption></figcaption></figure>

In a multi-bit adder, $$C_{\text{out}}$$ is added or _carried in_ to the next most significant bit (or into the next column). However, the half adder lacks a $$C_{\text{in}}$$ input to accept $$C_{\text{out}}$$ of the previous column. The _full adder_, described in the next section, solves this problem.

### Full Adder

A _full adder_ accepts the carry in $$C_{\text{in}}$$ as shown in Figure 5.3. The figure also shows the output equations for $$S$$ and $$C_{\text{out}}$$.

<figure><img src="../../.gitbook/assets/1-bit-full-adder.png" alt="" width="304"><figcaption></figcaption></figure>

### Carry Propagate Adder

An N-bit adders sums two N-bit inputs, A and B, and a carry in $$C_{\text{in}}$$ to produce an N-bit result $$S$$ and a carry out $$C_{\text{out}}$$. It is commonly called a _carry propagate adder_ (CPA) because the carry out of one bit propagates into the next bit. The symbol for CPA is shown in Figure 5.4; it is drawn just like a full adder except that A, B, and S are buses rather than single bits. Three common CPA implementations are called

1. [Ripple-carry adders](arithmetic-circuits.md#ripple-carry-adder)
2. [Carry-lookahead adders](arithmetic-circuits.md#carry-lookahead-adder)
3. Prefix adders (FYI)

<figure><img src="../../.gitbook/assets/carry-propagate-adder.png" alt=""><figcaption></figcaption></figure>

#### Ripple-Carry Adder

The simplest way to build an N-bit carry propagate adder is to chain together N full adders. The $$C_{\text{out}}$$ of one stage acts as the $$C_{\text{in}}$$ of the next stage, as shown in Figure 5.5 for 32-bit addition. This is called a _ripple-carry adder_.

<figure><img src="../../.gitbook/assets/32-bit-ripple-carry-adder.png" alt=""><figcaption></figcaption></figure>

The ripple-carry adder has the disadvantage of being **slow** when N is large. As $$S_{31}$$ depends on $$C_{30}$$, which depends on $$C_{29}$$, and so forth all the way back to $$C_{\text{in}}$$. We say that the carry _ripples_ through the carry chain. The delay of the adder, $$t_{\text{ripple}}$$ grows directly with the number of bits, as given in Equation 5.1, where $$t_{\text{FA}}$$ is the delay of a full adder,

$$
t_{\text{ripple}}=Nt_{\text{FA}}
$$

> In Ripple-Carry Adder, the main disadvantage is that the **carry** propagates very slow. Can we accelerate the process of knowing the carry of an adder quicker? Here comes the **carry-lookahead adder**.

#### Carry-Lookahead Adder

A _carry-lookahead_ adder (CLA) is another type of carry propogate adder that solves the problem caused by ripple-carry adder by dividing the adder into _blocks_ and providing circuitry to quickly determine the **carry out** of a block[^1] as soon as the [**carry in**](#user-content-fn-2)[^2] is known. Thus it is said to _look ahead_ across the blocks rather than waiting to ripple through all the full adders inside a block.

CLAs use _generate_ (G) and _propogate_ (P) signals that describe how a column or block determines the carry out.

* The $$i$$-th column of an adder is said to _generate_ a carry if it produces a carry out independent of the carry in. Hence, we have $$G_i=A_iB_i$$.
* The $$i$$-th column of an adder is said to _propagate_ a carry it is produces a carry [whenever there is a carry in](#user-content-fn-3)[^3]. Thus, we have $$P_i=A_i+B_i$$.

{% hint style="success" %}
One great advantage here is that both $$G_i$$ and $$P_i$$ depend only on the two inputs number $$A_i$$ and $$B_i$$, instead of the need of knowing the "carry-in" ($$C_{i-1}$$) of that block.
{% endhint %}

Using the $$G_i$$ and $$P_i$$, we can rewrite the carry out for each block of full adder ($$C_i$$)

$$
\begin{align*}
C_1&=p_0C_0+g_0 \\
C_2&=p_1C_1+g_1=p_1p_0C_0+p_1g_0+g_1\\
\cdots\\
C_i&=p_iC_{i-1}+g_i
\end{align*}
$$

Let's say we want to build a 32-bit carry-lookahead adder. Doing the above manipulation for 32 stages is cumbersome. However, we can do four stages first (a.k.a, implement a 4-bit carry-lookahead adder first as the fundamental block).

<figure><img src="../../.gitbook/assets/cg3207-lec04-4-bit-carry-lookahead-adder.png" alt="" width="563"><figcaption></figcaption></figure>

Using this fundamental block (4-bit carry-lookahead adder), we can build bigger adders. Figure 5.6 (a) shows a 32-bit carry-lookahead adder composed of **eight** 4-bit blocks. Each block contains a 4-bit ripple-carry adder and some lookahead logic to compute the carry out of the unit given the carry in, as shown in Figure 5.6 (b).&#x20;

<figure><img src="../../.gitbook/assets/32-bit-carry-lookahead-adder.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
#### Image Explanation

1. The AND and OR gates needed to compute the single-bit generate and propagate signals, $$G_i$$ and $$P_i$$, from $$A_i$$ and $$B_i$$ are left out for brevity.
2. The block propagate ($$P_i$$) and generate ($$G_i$$) signals for 4-bit blocks ($$i$$ refers to the block number here),
   1. $$P_0=p_3p_2p_1p_0$$, $$P_0$$ is the same as $$P_{[3:0]}$$ shown in the figure above
   2. $$G_0=g_3+p_3(g2+p2(g1+p1g_0))$$, $$G_0$$ is the same as $$G_{[3:0]}$$ shown in the figure above.
3. The ripple effect is avoided within a unit, but the ripple carry effect will be present between units. Nevertheless, the delays are significantly lowe than the orginal ripple carry adder.
{% endhint %}

### Putting it all together

HDLs provide the `+` operation to specify a CPA. HDL Example 5.1 describes a CPA with carries in and out

{% tabs %}
{% tab title="SystemVerilog" %}
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
{% endtab %}

{% tab title="Verilog" %}
{% code title="Example 5.1 Adder" lineNumbers="true" %}
```verilog
module adder #(parameter N = 8)
              (input  [N-1:0] a, b,
               input          cin,
               output [N-1:0] s,
               output         cout);
  assign {cout, s} = a + b + cin;
endmodule
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Subtraction

Recall from [previous part](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#signed-binary-number-representation) that adders can add positive and negative numbers using [two's complement number representation](https://wenbo-notes.gitbook.io/ddca-notes/textbook/from-zero-to-one/number-systems#id-2s-complement). Substraction is almost as easy:

1. flip the sign of the second number
2. then add it

{% hint style="info" %}
Flipping the sign of a two's complement number is done by inverting the bits and adding 1.
{% endhint %}

Thus, subtraction is performed with a single CPA by adding $$A+\bar B$$ with $$C_{\text{in}}=1$$. Figure 5.9 shows the symbol for a subtractor and the underlying hardware for performing $$Y=A-B$$.&#x20;

<figure><img src="../../.gitbook/assets/n-bit-subtractor.png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
An N-bit subtractor needs **N NOT gates**. This is tricky! Don't just see from the figure and then deduce that only one NOT gate is needed!
{% endhint %}

HDL Example 5.2 describes a subtractor.

{% tabs %}
{% tab title="SystemVerilog" %}
{% code title="Example 5.2 Subtractor" lineNumbers="true" %}
```verilog
module subtractor #(parameter N = 8)
                   (input  logic [N-1:0] a, b,
                    output logic [N-1:0] y);
  assign y = a - b;
endmodule
```
{% endcode %}
{% endtab %}

{% tab title="Verilog" %}
{% code title="Example 5.2 Subtractor" lineNumbers="true" %}
```verilog
module subtractor #(parameter N = 8)
                   (input  [N-1:0] a, b,
                    output [N-1:0] y);
  assign y = a - b;
endmodule
```
{% endcode %}
{% endtab %}
{% endtabs %}

### Signed Addition/Subtraction

In the signed addition/subtraction, we have four flages, N(egative), Z(ero), C(array), V(Overflow), to indicate the status of our result. The following part will explain when certain flag is set,

{% hint style="warning" %}
Before that, let's make some clarifications or conventions first.

1. We denote the LSB at bit 0 or 0-th bit, so on and so forth for all the remaining bits.
2. When we add two N-bit numbers, we will get a N+1 bit number. However, we treat the MSB (the N-th bit) as the **carry bit**, and the remaining N bits are the **result**. So, again when we say the MSB of the **result**, we are referring to the (N-1)-th bit actually.
{% endhint %}

#### Signed Addition

<figure><img src="../../.gitbook/assets/cg3207-lec04-signed-addition.png" alt="" width="563"><figcaption></figcaption></figure>

By observing the addition examples above, we may find out that

1. For _unsigned addition_, carry=1 indicates that the result is wrong.
2. For _signed addition_,
   1. Overflow occurs when (+ve) + (+ve) = (-ve) or (-ve) + (-ve) = (+ve), a.k.a, two negative number addition gives out a positive number or two positive number addition gives a negative number
      1. To determine the overflow flag, we always treat the two operands as **signed**! So, this process is just to see that if the sign bits (MSB) of the **operands** are the same, and is different from that of the **result** (the (N-1)-th bit), there is an overflow.
      2. Overflow can never occur when you add two numbers of different signs
      3. Carry and overflow are **unrelated** (However, carry is sometime called "unsigned overflow")
   2. Carry flag is set when the value in the <mark style="color:blue;">blue rectangle</mark> is one.
   3. Negative flag is determined by only looking at the MSB of the result. (the (N-1)-th bit actually)
   4. Zero flag is determined by only looking at the N-bit result.

{% hint style="success" %}
#### Notes

1. In addition, there are some small tips
   1. N flag and Z flag cannot be 1 simultaneously.
   2. Z flag and V flag can be set simultaneously. e.g., 1000+1000=0000 with carry out 1, and the interpretation of the result to be 0.
2. The hardware as well as the instruction used for performing both signed and unsigned addition are the same
   1. The hardware doesn't know and doesn't care if we are dealing with signed or unsigned.
   2. Only the _interpretation_ of the result and/or the way the _condition codes_ are used are different.
   3. So, first use signed/unsigned to write out the two operands, then just do the binary addition on these two operands and derive the flags.
{% endhint %}

#### Signed Subtraction

<figure><img src="../../.gitbook/assets/cg3207-lec04-signed-subtraction.png" alt="" width="563"><figcaption></figcaption></figure>

By observing the subtraction examples above, we can find out that

1. For _unsigned_ subtraction, **borrow** indicates that the result is wrong.
   1. In ARM, **carry** is just **NOT borrow**. However, in Intel, **carry** is **borrow**, but in subtraction, it will flip the carry bit to get the carry flag when it is subtraction.
2. For _signed_ subtrcation,
   1. Overflow occurs at when (+ve) - (-ve) = (-ve) or (-ve) - (+ve) = (+ve).
      1. And similar to the signed addition, we always treat the two operands as **signed**. So, if the sign bits (MSB) of the **operands** are different, and the **result** has a sign same as that of the second operand, there is an overflow.
      2. Overflow can never occur in subtraction when operands are of the **same sign**.
   2. Carry flag is set when you are looking at the hardware version. Or the flip the borrow in pen\&paper version.

{% hint style="success" %}
#### Notes

1. The hardware as well as the instruction used for performing both signed and unsigned subtraction are the same
   1. The hardware doesn't know and doesn't care if we are dealing with signed or unsigned.
   2. Only the _interpretation_ of the result and/or the way the _condition codes_ are used are different.
   3. So, first use signed/unsigned to write out the two operands, then just do the binary subtraction on these two operands and derive the flags.
{% endhint %}

***

In summary, we know the following points from the human interpretation side,

1. When C = 1, it indicates the **unsigned addition result** is wrong.
2. When C = 0, it indicates that the **unsigned subtraction** **result** is wrong.
3. When V = 1, it indicates that both **signed addition** and **signed subtraction** result are wrong.

***

For RISC-V, NZCV is **irrelevant** only when ALU does the **addition**. But, it is still recommended to go through how each flag of the NZCV is generated during [addition](arithmetic-circuits.md#signed-addition).

Instead, in RISC-V, NZCV is **relevant** when ALU does the **subraction**. Again, it is important to go through how NZCV is generated during [subtraction](arithmetic-circuits.md#signed-subtraction). In RISC-V, the subtraction can be done in

* `sub`
* branch/slt variants

In pure `sub` instruction, the NZCV may not be that useful. But in branch/slt variants, this NZCV flag, which is not stored as bits for use by future instructions, becomes much more useful. The different comparisons, like `eq`, etc, are implemented using these NZCV flags.

| Comparison | Signed/Unsigned | Condition | Implemented as | Uses                  |
| ---------- | --------------- | --------- | -------------- | --------------------- |
| eq         | Both            | A == B    | Z              | Z flag                |
| ne         | Both            | A ≠ B     | ¬Z             | Z flag                |
| lt         | Signed          | A < B     | N ⊕ V          | Signed overflow logic |
| ge         | Signed          | A ≥ B     | ¬(N ⊕ V)       | Signed overflow logic |
| ltu        | Unsigned        | A < B     | ¬C             | Borrow logic          |
| geu        | Unsigned        | A ≥ B     | C              | Borrow logic          |

{% hint style="success" %}
#### Table Explanation

1. For the `lt` and `ge`, whch are for signed numbers, we must first make sure there is **no overflow**, then check the N flag.
2. For the `ltu` and `geu`, use the ARM borrow logic to understand. If `A<B`, then A-B will need a borrow, thus the carry flag is 0, and vice versa.
3. `sltu` is useful to check whether the subtraction of two numbers will need a **borrow** or not.
{% endhint %}

<details>

<summary>Multi-word arithmetic in RISC-V</summary>

Suppose we want to do 64-bit addition and subtraction in RISC-V. We use `[t0:t1]` to store the 64-bit number A and `[t2:t3]` to store the 64-bit number B. And `[t4:t5]` to store the final result.

For the addition, we have,

{% code lineNumbers="true" %}
```armasm
add   t4, t0, t2        # low  word sum
sltu  t6, t4, t0        # carry = (sum < t0)
add   t5, t1, t3        # high word sum
add   t5, t5, t6        # add carry
```
{% endcode %}

For the subtraction, we have,

{% code lineNumbers="true" %}
```armasm
sub   t4, t0, t2        # low  word diff
sltu  t6, t0, t2        # borrow = (t0 < t2)
sub   t5, t1, t3        # high word diff
sub   t5, t5, t6        # add carry
```
{% endcode %}

So, in summary, we have

| Operation   | Condition for carry/borrow | Equivalent test            | RISC-V instruction |
| ----------- | -------------------------- | -------------------------- | ------------------ |
| Addition    | carry                      | `(sum < A)` or `(sum < B)` | `sltu t6, sum, t0` |
| Subtraction | borrow                     | `(A < B)`                  | `sltu t6, t0, t2`  |

</details>

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

{% hint style="success" %}
Here, the \[N-1] means the output is the $$(N-1)^{\text{th}}$$ bit of the result N.&#x20;
{% endhint %}
{% endstep %}
{% endstepper %}

HDL Example 5.3 shows how to ues various comparison operations.

{% tabs %}
{% tab title="SystemVerilog" %}
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
{% endtab %}

{% tab title="Verilog" %}
{% code title="Example 5.3 Comparators" lineNumbers="true" %}
```verilog
module comparators #(parameter N = 8)
                    (input  [N-1:0] a, b,
                     output         eq, neq,
                     output         lt, lte,
                     output         gt, gte);
  assign eq  = (a == b);
  assign neq = (a != b);
  assign lt  = (a < b);
  assign lte = (a <= b);
  assign gt  = (a > b);
  assign gte = (a >= b);
endmodule
```
{% endcode %}
{% endtab %}
{% endtabs %}

## ALU

An _Arithmetic/Logical Unit_ (ALU) combines a variety of mathematical and logical operations into a single unit. For example, a typical ALU might perform addition, subtraction, magnitude comparison, AND, and OR operations. The ALU forms the heart of most computer systems.

Figure 5.14 shows the symbol for an N-bit ALU with N-bit inputs and outputs.

<figure><img src="../../.gitbook/assets/alu-symbol.png" alt=""><figcaption></figcaption></figure>

The ALU receives a control signal F that specifies which function to perform. Table 5.1 lists typical functions that the ALU can perform.

<figure><img src="../../.gitbook/assets/alu-operations.png" alt="" width="394"><figcaption></figcaption></figure>

Figure 5.15 shows an implementation of the ALU.

<figure><img src="../../.gitbook/assets/n-bit-alu.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
#### Image Explanation

1. $$F_2$$ is also the carry in to the adder.
2. In two's complement arithmetic, $$\bar B+1=-B$$.
3. When $$F_{\text{[2:0]}}=111$$, the ALU performs the _set if less than_ (SLT) operation. When $$A<B$$, $$Y=1$$. Otherwise, $$Y=0$$. In other words, Y is set to 1 if A is less than B.
   1. SLT is performed by computing `S=A-B`. If S is negative (e.g., the sign bit is set), A is less than B. The _zero extend unit_ produces an N-bit output by concatenating is 1-bit input with 0's in the most significant bits. The sign bit ( $$(N-1)^{\text{th}}$$ bit) of S is the input to the zero extend unit.
4. The AND and OR gate shown in the figure above both need **32 2-input** AND/OR gate to implement! **NOT One!**
5. The multiplexer to select whether input is the N-bit $$\bar B$$ or $$B$$ can be implemented using N 2-input [XOR gate](https://wenbo-notes.gitbook.io/ddca-notes/lab/resources/verilog-lifesaver#hint), where one input of it is the 1-bit from the N-bit $$B$$, the other is from $$F_2$$.
{% endhint %}

Some ALUs produce extra outputs, called _flags_, that indicate information about the ALU output. For example, an _overflow flag_ indicates that the result of the adder overflowed. A _zero flag_ indicates that teh ALU output is 0.

The HDL for an N-bit ALU is shown as follows

{% tabs %}
{% tab title="SystemVerilog" %}
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
{% endtab %}

{% tab title="Verilog" %}
> **TODO:** Wait for adding LOL.
{% endtab %}
{% endtabs %}

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

{% tabs %}
{% tab title="SystemVerilog" %}
{% code title="Example 5.4 Multiplier" lineNumbers="true" %}
```verilog
module multiplier #(parameter N = 8)
                   (input  logic [N-1:0]    a, b,
                    output logic [2*N-1:0] y);
  assign y = a * b;
endmodule
```
{% endcode %}
{% endtab %}

{% tab title="Verilog" %}
{% code title="Example 5.4 Multiplier" lineNumbers="true" %}
```verilog
module multiplier #(parameter N = 8)
                   (input  [N-1:0]    a, b,
                    output [2*N-1:0] y);
  assign y = a * b;
endmodule
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Division

> **TODO:** FYI part first.

[^1]: Here, one block is a one-bit full adder.

[^2]: Here, the “carry-in” refers to the **initial** carry input of the adder. Each adder has only one carry-in signal; for multi-bit adders, this single carry-in propagates through the stages of the adder to produce carries for subsequent bits.

[^3]: This means, when there is a carry in, the column will always produce a carry out, which **propagates** the carry in it receives.
