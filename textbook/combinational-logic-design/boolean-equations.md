# Boolean Equations

## Terminology

* A _minterm_ is a product involving **all** of the inputs to the function.&#x20;
* A _maxterm_ is a sum involving **all** of the inputs to the function.

e.g., $$A\bar B\bar C$$ is a minterm, $$A+\bar B+C$$ is a maxterm for a function of the three variables A, B, C.

## Sum-of-Products Form

We can write a Boolean equation for any truth table by summing each of the minterms for which the output is TRUE. And this boolean equation is called the _sum-of-products canonical form_ of a function because it is the sum (OR) of products (ANDs forming minterms)

## Product-of-Sums Form

An **alternative** way of expressing Boolean functions is the _product-of-sums canonical form_. Each row of a truth table corresponds to a maxterm that is FALSE for that row.

<details>

<summary>SoP and PoS, which one to choose?</summary>

It depends on there are how many TRUEs and FALSEs in your truth table.

Sum-of-products (SoP) produces the shortest equations when the output is TRUE on only a few rows of a truth table; product-of-sums (PoS) is simpler when the output is FALSE on only a few rows of a truth table.

</details>
