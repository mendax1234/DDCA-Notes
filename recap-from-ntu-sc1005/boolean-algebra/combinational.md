# Combinational

## Designing and Implementing Combinational Logic Circuit

1. The function of a required logic circuit can be fully described by a truth table.
2. To design the circuit, we obtain the **Boolean expression** from the truth table.
3. The Boolean expression can then be implemented using a proper choice of logic gates.

### Forms of Boolean Expressions

There are two forms of Boolean Expressions:

{% columns %}
{% column %}
<p align="center"><strong>Canonical Form</strong></p>

* Sum of minterms expressions (SOm)
* Product of Maxterms expression (POM)
{% endcolumn %}

{% column %}
<p align="center"><strong>Standard Form</strong></p>

* Sum of products expression (SOP)
* Product of sums expressions (POS)
{% endcolumn %}
{% endcolumns %}

***

A SOm is **always** a SOP, but a SOP **may not** be a SOm. Simiarly, a POM is **always** a POS, but a POS **may not** be a POM.

{% hint style="warning" %}
Usually, we are interested in finding the **simplest standard form** for a combinational logic circuit. So, the canonical form is often considered as a "middle" step for us to find the simplest standard form by using kinds of different methods, like

1. Algebraic method
2. Karnaugh map (K-map)
3. etc
{% endhint %}

### Obtain Boolean Expressions from truth table

{% stepper %}
{% step %}
**Observation**

This method can be used when the circuit isn't too difficult.
{% endstep %}

{% step %}
**K-map**

**Steps** to use K-map to obtain the simplest standard form

1. Form the K-map table by observing the truth table of the logic circuit
2. Find Loops in the K-map table
   1. Loop 1 to obtain **SOP** expression.
   2. Loop 0 to obtain **POS** expression.

{% hint style="warning" %}
#### **Things to notice with K-map**

1. Only  $$2^n$$ number of "neighboring" 0/1 can be looped together.
2. No looping along **diagnoal**.
3. **All** 0/1 must be looped.
4. Use the **biggest loops** and the **fewest loops**
5. A **square** may be looped more than once.
6. "Don't cares" should only be looped if it helps to simplify a Boolean Expression (e.g., helps to form a bigger loop)&#x20;
{% endhint %}

**Summary** of K-map

1. In the test, we are usually given the behavior of a logic circuit, this is equal to giving us the **truth** table of one logic circuit.
2. With the truth table, we can write down the SOm and POM of this logic circuit very quickly, just by observation.
3. However, SOm and POM are not what we want. Thus we can use different methods, like K-map, etc, to obtain the **simplest standard form** from the truth table / the behavior of a logic circuit.
{% endstep %}
{% endstepper %}
