# Lab 02 - Single Cylce RV Processor

## Introduction

In this lab, we will build a single-cycle RISC-V processor and the lab manual is [here](https://nus-cg3207.github.io/labs/asst_manuals/Asst_02/Asst_02/). And our processor will support the followig instructions for now,

* `add`, `addi`, `sub`, `and`, `andi`, `or`, `ori`
* `lw`, `sw`
* `beq`, `bne`, `jal` (without linking, that is, without saving the return address).&#x20;
* `lui`,`auipc`
* `sll`, `srl`, `sra`

## Design Files

### Why does the Wrapper exist?

`Wrapper.v` is long, but it’s basically a **simulation harness** that sits between your RISC-V core (RV) and the outside world (testbench or FPGA board). It provides memories and **memory-mapped peripherals** so that the processor can interact with LEDs, DIP switches, buttons, UART, OLED, etc. You can think of it as the “motherboard” that your CPU plugs into.

#### Purpose of Wrapper

* Provides **IROM (instruction memory)** and **DMEM (data memory)** to the processor.
* Provides **MMIO (memory-mapped I/O)** registers for peripherals like LEDs, UART, DIP switches, OLED, accelerometer, etc.
* Translates “clean, parallel” signals into abstract peripherals that are easy to monitor in simulation.
* Used only for **simulation** (not synthesis directly). On the FPGA, the higher-level `TOP.vhd` connects the wrapper to actual hardware pins.

So, it makes your RISC-V behave like it’s running on a small SoC with peripherals.

#### Interfaces (ports)

Look at the top port list:

* **Inputs:**
  * `DIP` → DIP switch values (like user input).
  * `PB` → Pushbuttons (3 bits).
  * `UART_RX`, `UART_RX_valid` → incoming UART data from PC/testbench.
  * `ACCEL_Data`, `ACCEL_DReady` → accelerometer values (if connected).
  * `RESET`, `CLK`.
* **Outputs:**
  * `LED_OUT` → control LEDs.
  * `LED_PC` → debug: shows current PC\[8:2].
  * `SEVENSEGHEX` → 32-bit value mapped to 7-seg display.
  * `UART_TX`, `UART_TX_valid` → outgoing UART data.
  * `UART_RX_ack` → handshake back to UART.
  * `OLED_Write`, `OLED_Row`, `OLED_Col`, `OLED_Data` → OLED interface.

So: input = board/testbench → Wrapper → CPU; output = CPU → Wrapper → board/testbench.

#### Memories and Addresses

Wrapper defines 3 spaces:

* **IROM**: instruction memory (program comes from `.mem` file dumped by RARS).
* **DMEM**: data memory (for global variables, stack, etc.).
* **MMIO**: memory-mapped I/O (LEDs, switches, UART, OLED, etc.).

Memory map (important):

* `IROM_BASE = 0x0040_0000` → code section.
* `DMEM_BASE = 0x1001_0000` → data section.
* `MMIO_BASE = 0xFFFF_0000` → peripherals (LED, DIP, UART, etc.).

This matches what you configure in RARS.

#### Address Decode

When CPU issues a memory access (via `ALUResult`):

1. **Wrapper checks address range**
   * If `ALUResult` ∈ DMEM range → access Data Memory.
   * Else if `ALUResult` ∈ MMIO range → go to case statement.
2. **Case statement for MMIO**
   * Each peripheral is mapped to a fixed offset.
   * Wrapper generates a **decoded enable signal** (`dec_*`) for the right peripheral.
3. **Action depends on** `MemRead`/`MemWrite_out`
   * If `MemWrite_out` **= 1** → Wrapper forwards `WriteData_out` to peripheral’s register.
   * If `MemRead` **= 1** → Wrapper drives `ReadData_in` with peripheral’s data.

{% code overflow="wrap" lineNumbers="true" %}
```verilog
if (ALUResult[31:DMEM_DEPTH_BITS] == DMEM_BASE[31:DMEM_DEPTH_BITS]) dec_DMEM <= 1'b1;
else if (ALUResult[31:MMIO_DEPTH_BITS] == MMIO_BASE[31:MMIO_DEPTH_BITS]) begin
    case (ALUResult[MMIO_DEPTH_BITS-1:2])
        UART_RX_VALID_OFF[MMIO_DEPTH_BITS-1:2]: dec_UART_RX_VALID <= 1'b1;
        UART_RX_OFF[MMIO_DEPTH_BITS-1:2]: dec_UART_RX <= 1'b1;
        UART_TX_READY_OFF[MMIO_DEPTH_BITS-1:2]: dec_UART_TX_READY <= 1'b1;
        UART_TX_OFF[MMIO_DEPTH_BITS-1:2]: dec_UART_TX <= 1'b1;
        OLED_COL_OFF[MMIO_DEPTH_BITS-1:2]: dec_OLED_COL <= 1'b1;
        OLED_ROW_OFF[MMIO_DEPTH_BITS-1:2]: dec_OLED_ROW <= 1'b1;
        OLED_DATA_OFF[MMIO_DEPTH_BITS-1:2]: dec_OLED_DATA <= 1'b1;
        OLED_CTRL_OFF[MMIO_DEPTH_BITS-1:2]: dec_OLED_CTRL <= 1'b1;
        ACCEL_DATA_OFF[MMIO_DEPTH_BITS-1:2]: dec_ACCEL_DATA <= 1'b1;
        ACCEL_DREADY_OFF[MMIO_DEPTH_BITS-1:2]: dec_ACCEL_DREADY <= 1'b1;
        LED_OFF[MMIO_DEPTH_BITS-1:2]: dec_LED <= 1'b1;
        DIP_OFF[MMIO_DEPTH_BITS-1:2]: dec_DIP <= 1'b1;
        PB_OFF[MMIO_DEPTH_BITS-1:2]: dec_PB <= 1'b1;
        SEVENSEG_OFF[MMIO_DEPTH_BITS-1:2]: dec_SEVENSEG <= 1'b1;
        CYCLECOUNT_OFF[MMIO_DEPTH_BITS-1:2]: dec_CYCLECOUNT <= 1'b1;
        default: bad_MEM_addr <= 1'b1;
    endcase
end else bad_MEM_addr <= 1'b1;

assign dec_MMIO_read = MemRead || dec_DIP || dec_PB || dec_UART_RX_VALID || dec_UART_RX || dec_UART_TX_READY || dec_UART_TX || dec_CYCLECOUNT || dec_ACCEL_DATA || dec_ACCEL_DREADY ;
assign MemWrite = MemWrite_out[3] || MemWrite_out[2] || MemWrite_out[1] || MemWrite_out[0];

//----------------------------------------------------------------
// Delaying the decoded signals for multiplexing (delay only if using synch read for memory)
//----------------------------------------------------------------
always @(*) begin  // @posedge CLK only if using synch read for memory
    dec_DMEM_W <= dec_DMEM;
    dec_MMIO_read_W <= dec_MMIO_read;
end

//----------------------------------------------------------------
// Input (into RV) multiplexing
//----------------------------------------------------------------
always @(*) begin
    if (dec_DMEM_W) ReadData_in <= ReadData_DMEM;
    else  // dec_MMIO_read_W
        ReadData_in <= ReadData_MMIO;
end
```
{% endcode %}

> But why got delayed here?

#### I/O Multiplexing

CPU only sees **one** `ReadData_in` **bus**. Wrapper multiplexes between:

* DMEM read data (`ReadData_DMEM`).
* MMIO read data (`ReadData_MMIO`).

{% code lineNumbers="true" %}
```verilog
//----------------------------------------------------------------
// Input (into RV) multiplexing
//----------------------------------------------------------------
always @(*) begin
    if (dec_DMEM_W) ReadData_in <= ReadData_DMEM;
    else  // dec_MMIO_read_W
        ReadData_in <= ReadData_MMIO;
end
```
{% endcode %}

Same for writes: if CPU does `sw x1, LED_OFF(s0)`, Wrapper routes the data to LED register.

#### Peripherals Implementations

* **LEDs (**`LED_OFF`**)**: writing updates `LED_OUT`.
* **DIP (**`DIP_OFF`**)**: reading returns DIP switch positions.
* **PushButtons (**`PB_OFF`**)**: reading returns button states.
* **SevenSeg (**`SEVENSEG_OFF`**)**: writing updates 7-seg display.
* **UART**:
  * `UART_TX_OFF`: write to send a char (if ready).
  * `UART_RX_OFF`: read to get received char.
  * Has handshake signals (`UART_TX_ready`, `UART_RX_valid`).
* **OLED**: more complex, supports different color modes and auto-incrementing row/col.
* **CycleCounter (**`CYCLECOUNT_OFF`**)**: free-running cycle counter (useful for performance measurement).
* **Accelerometer**: reads `ACCEL_Data`.

#### Connection to RV

Here, please go back to the Lec 03's microarchitecture so that you can better understand what's going on here.

{% stepper %}
{% step %}
#### Instruction fetch path

* CPU outputs **PC**.
* Wrapper uses PC to fetch the **Instr** from instruction memory.
* Wrapper sends **Instr** back into CPU.

So CPU always gets the next instruction via this loop.
{% endstep %}

{% step %}
#### Data access path

The data access path has two parts, one is to load data from either DMEM or MMIO, the other is to store dat to either DMEM or MMIO.

1. **Load**: Loading from both RAM and peripherals look identical to CPU, only Wrapper decides the source.
   * CPU computes address in ALU → **ALUResult**.
   * CPU asserts **MemRead = 1**. (This is not available in our Lec 03 microarchitecture)
   * Wrapper checks ALUResult:
     * If address ∈ **DMEM range** → fetch from RAM.
     * If address ∈ **MMIO range** → fetch from peripheral (e.g. DIP switch value).
   * Wrapper puts that value onto **ReadData\_in**.
   * CPU reads **ReadData\_in** and writes into register file.
2. **Store**: Storing either update memory or trigger side effects on peripherals.
   * CPU computes address in ALU → **ALUResult**.
   * CPU outputs the store value → **WriteData\_out**.
   * CPU asserts **MemWrite\_out = 1**. (This is the `MemWrite` on our Lec 03 microarchitecture)
   * Wrapper checks ALUResult:
     * If address ∈ **DMEM range** → write WriteData\_out into RAM.
     * If address ∈ **MMIO range** → forward WriteData\_out into peripheral (e.g. set LEDs, update 7-seg).
   * CPU does not expect anything on ReadData\_in.

So, we can have a clear picture that

* `ReadData_in` = data bus from memory/peripherals back to CPU (for loads).
* `MemWrite_out` = control signal telling wrapper “do a store now,” with data on `WriteData_out`.
{% endstep %}
{% endstepper %}

## Tips

### Demo

As you may be confused about what you are going to demo, here are some tips from the teaching team

{% stepper %}
{% step %}
#### General Caveats

1. Make your output **dependant on** some user inputs. A.k.a, don't hardcode the output using a counter, etc.
2. Design your assembly program such that if one instruction doesn't work, the final output will be completely different. In other words, to get this output correctly given this input, every instruction in your assembly program should work correctly.
   1. **Don't just look at the final output** to state that every instruction of your prgram works. For example, if you shift right by 5 bits and shift left by 5 bits, even if your shift instructions don't work, your result will still be right.
{% endstep %}

{% step %}
#### Testbench

Your progam should run based on your user input, so the testbench just serves as **simulating** that user input and check whether the output of your program is correct or not.
{% endstep %}

{% step %}
#### More about OLED

In the lab, OLED is provided, but it is just another output. The fancy demo you may see from the lab by Dr. Rajesh is just another way to craft the **well-designed** assembly code.
{% endstep %}

{% step %}
#### General Steps

{% hint style="success" %}
This is the most important tip for demo!
{% endhint %}

1. Assembly code correctness: Step through the assembly program step by step to make sure the algorithm works correctly on the "high-level".
2. HDL (Vivado) simulation: Step through the each instruction to make sure each instruction is executed correctly on vivado simulation (behavioral + post-synthesis)
3. Hardware demo: This step **cannot step through** and thus we just provide the input and see the output.
{% endstep %}
{% endstepper %}
