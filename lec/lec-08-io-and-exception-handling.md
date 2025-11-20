# Lec 08 - IO and Exception Handling

## Basic I/O Concepts

**Input/Output (I/O)** systems are used to connect a computer with external devices called **peripherals**. In a personal computer, the devices typically include keyboards, monitors, printers, and wireless networks. In **embedded systems**, devices could include a toaster’s heating element, a doll’s speech synthesizer, an engine’s fuel injector, a satellite’s solar panel positioning motors, and so forth. A processor accesses an **I/O device** using the **address** and **data busses** in the same way that it accesses memory.

### Memory-Mapped I/O

A portion of the **address space** is dedicated to **I/O devices** rather than memory. For example, suppose that **physical addresses** in the range `0x20000000` to `0x20FFFFFF` are used for I/O. Each **I/O device** is assigned one or more memory addresses in this range.

* A **store** (`sw`) to the specified address sends data to the device.
* A **load** (`lw`) receives data from the device.&#x20;

This method of communicating with **I/O devices** is called **memory-mapped I/O**.

{% hint style="success" %}
Software that communicates with an **I/O device** is called a **device driver**. For example, you might have downloaded or installed device drivers for your printer or other I/O device.
{% endhint %}

#### Hardware

In a system with **memory-mapped I/O**, a **load** or **store** may access either memory or an **I/O device**. The following figure shows the hardware needed to support two memory-mapped I/O devices.

<figure><img src="../.gitbook/assets/memory-mapped-io-hardware.png" alt=""><figcaption><p> Support hardware for memory-mapped I/O</p></figcaption></figure>

An **address decoder** determines which device communicates with the processor. It uses the **Address** and **MemWrite** signals to generate control signals for the rest of the hardware. The **ReadData multiplexer** selects between memory and the various **I/O devices**. **Write-enabled registers** hold the values written to the **I/O devices**.

{% hint style="success" %}
This is technically the working principle of our [`Wrapper.v`](../lab/lab-02-single-cylce-rv-processor.md#purpose-of-wrapper) used in our Labs!
{% endhint %}
