# Sequential Logic Design

As we have seen a bit earlier in the [combinational-logic-design](../combinational-logic-design/ "mention"). The outputs of sequential logic depend on both current and prior input values. Hence, sequential has **memory**. This memory is called the state of the logic.

<details>

<summary>What does memory exactly mean?</summary>

TLDR, after we learn the [D Flip-flop](https://wenbo-notes.gitbook.io/ddca-notes/textbook/sequential-logic-design/latches-and-flip-flops#d-flip-flop), we will see that _memory_ means it can **remember** what the previous state was until there is a clock edge.

</details>

Sequential logic might explicitly remember certain previous inputs, or it might distill the prior inputs into a smaller amount of information called the _state_ of the system. The state of a digital sequential circuit is a set of bits called _state variables_ that contain all the information about the past necessary to explain the future behavior of the state.

We will begin by studying latches and flip-flops, which are simple sequential circuits that store **one bit** of state.
