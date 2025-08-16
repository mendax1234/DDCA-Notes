# A Small Recap

From this chapter, bascially we've seen how our logic gates are built and worked from the **Physics Level** and **Digital Circuits** Level. To sum up, we can start by going through what actually happens under the hood.

## Physics Level - Voltage

A digital logic gate has four main parts: 1) input, 2) output, 3) supply voltage, and 4) GND. Technically, all these four parts will have a certain voltage. And you may notice that

1. For Supply Voltage & GND: These two voltages by right won't change
2. For Input & Output: These two voltages are changing

We can use the NOT gate built with CMOS transistors as an example. When the voltage on the input is very small, and small to around 0. This voltage will turn on the pMOS transistor, thus the output will be pulled-up to the supply voltage (think of it as a process, not immediately pulled-up). And as we have seen that each wire has a capacitance, thus at this time, the current flows into the capacitance, thus the output voltage will increase. The increased voltage on this NOT gate's output can be then sensed by the next gate's input.

## Digital Circuits Level - 0 & 1

As we are clear what's happening at the physics level, from the digital level, these voltages are mapped to 0 & 1 according to the four logic leves we have seen in [#logic-levels](beneath-the-digital-abstraction.md#logic-levels "mention").
