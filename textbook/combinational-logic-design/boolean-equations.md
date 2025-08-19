# Boolean Equations

In this section, our purpose is to write a Boolean equation for any logic function given its truth table.

## Terminology

{% stepper %}
{% step %}
**Minterm and Maxterm**

* A _minterm_ is a product involving **all** of the inputs to the function.&#x20;
* A _maxterm_ is a sum involving **all** of the inputs to the function.

e.g., $$A\bar B\bar C$$ is a minterm, $$A+\bar B+C$$ is a maxterm for a function of the three variables A, B, C.

{% hint style="info" %}
There exists **one and only one** input value such that given minterm = 1 / maxterm = 0. This property ensures that when writing the boolean equation from a truth table:

1. For Sum of Product: We sum each of the minterms of which the output is TRUE.
2. For Product of Sum: We multiply each of the maxterms of which the output is FALSE.
{% endhint %}
{% endstep %}

{% step %}
**The order of oeprations**

The _order of operations_ is important when interpreting Boolean equations. In Boolean equations, NOT has the highest _precedence_, followed by AND, then OR.
{% endstep %}
{% endstepper %}

## Sum-of-Products Form

We can write a Boolean equation for any truth table by summing each of the minterms for which the output is TRUE. And this boolean equation is called the _sum-of-products canonical form_ of a function because it is the sum (OR) of products (ANDs forming minterms)

## Product-of-Sums Form

An **alternative** way of expressing Boolean functions is the _product-of-sums canonical form_. Each row of a truth table corresponds to a maxterm that is FALSE for that row.

<details>

<summary>SoP and PoS, which one to choose?</summary>

It depends on there are how many TRUEs and FALSEs in your truth table.

Sum-of-products (SoP) produces the shortest equations when the output is TRUE on only a few rows of a truth table; product-of-sums (PoS) is simpler when the output is FALSE on only a few rows of a truth table.

</details>
