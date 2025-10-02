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

`Wrapper.v` is long, but it’s basically a **simulation harness** that sits between our RISC-V core (RV) and the outside world (testbench or FPGA board). It provides memories and **memory-mapped peripherals** so that the processor can interact with LEDs, DIP switches, buttons, UART, OLED, etc. We can think of it as the “motherboard” that our RV CPU plugs into.

In short, `Wrapper.v` is just the Verilog implementation of our [Lec 03 microarchitecture](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-03-risc-v-isa-and-microarchitecture#support-for-lui-and-auipc) but adding the MMIO part. (Go to the [spoiler](lab-02-single-cylce-rv-processor.md#connection-to-rv) at the back)

#### Purpose of Wrapper

* Provides **IROM (instruction memory)** and **DMEM (data memory)** to the processor.
* Provides **MMIO (memory-mapped I/O)** registers for peripherals like LEDs, UART, DIP switches, OLED, accelerometer, etc.
* Translates “clean, parallel” signals into abstract peripherals that are easy to monitor in simulation.
* Used only for **simulation** (not synthesis directly). On the FPGA, the higher-level `TOP_Nexys.vhd` connects the wrapper to actual hardware pins.

So, it makes our RISC-V behave like it’s running on a small SoC with peripherals.

#### I/O Ports

<table><thead><tr><th width="153.5">Signal</th><th width="100">Direction</th><th width="93.5">Width</th><th>Description</th></tr></thead><tbody><tr><td><code>DIP</code></td><td>In</td><td>16</td><td>DIP switch inputs (not debounced).</td></tr><tr><td><code>PB</code></td><td>In</td><td>3</td><td>Push buttons (BTNL, BTNC, BTNR; not debounced).</td></tr><tr><td><code>LED_OUT</code></td><td>Out (reg)</td><td>8</td><td>LEDs [7:0] showing processor results.</td></tr><tr><td><code>LED_PC</code></td><td>Out</td><td>7</td><td>Shows <code>PC[8:2]</code> on LEDs [15:9].</td></tr><tr><td><code>SEVENSEGHEX</code></td><td>Out (reg)</td><td>32</td><td>Data for 8-digit 7-seg display.</td></tr><tr><td><code>UART_TX</code></td><td>Out (reg)</td><td>8</td><td>Byte sent to PC/testbench via UART.</td></tr><tr><td><code>UART_TX_ready</code></td><td>In</td><td>1</td><td>UART ready flag (ok to write new TX byte).</td></tr><tr><td><code>UART_TX_valid</code></td><td>Out (reg)</td><td>1</td><td>Indicates Wrapper wrote a new UART TX byte.</td></tr><tr><td><code>UART_RX</code></td><td>In</td><td>8</td><td>Byte received from PC/testbench via UART.</td></tr><tr><td><code>UART_RX_valid</code></td><td>In</td><td>1</td><td>Indicates new RX data available.</td></tr><tr><td><code>UART_RX_ack</code></td><td>Out (reg)</td><td>1</td><td>Acknowledge that RX byte was read.</td></tr><tr><td><code>OLED_Write</code></td><td>Out (reg)</td><td>1</td><td>Pixel update signal for OLED.</td></tr><tr><td><code>OLED_Col</code></td><td>Out (reg)</td><td>7</td><td>OLED column index.</td></tr><tr><td><code>OLED_Row</code></td><td>Out (reg)</td><td>6</td><td>OLED row index.</td></tr><tr><td><code>OLED_Data</code></td><td>Out (reg)</td><td>24</td><td>OLED pixel data <code>&#x3C;R,G,B></code> (8/8/8 aligned).</td></tr><tr><td><code>ACCEL_Data</code></td><td>In</td><td>32</td><td>Packed <code>&#x3C;Temp, X, Y, Z></code> from accelerometer.</td></tr><tr><td><code>ACCEL_DReady</code></td><td>In</td><td>1</td><td>Accelerometer data ready flag.</td></tr><tr><td><code>RESET</code></td><td>In</td><td>1</td><td>Active-high reset.</td></tr><tr><td><code>CLK</code></td><td>In</td><td>1</td><td>Clock input (divided clock shown on LED[8]).</td></tr></tbody></table>

#### Memories

In `Wrapper.v` (our motherboard), we not only have our CPU `RV.v`, but also three memory spaces:

* **IROM**: instruction memory (program comes from `.mem` file dumped by RARS).
* **DMEM**: data memory (for global variables, stack, etc.).
* **MMIO**: memory-mapped I/O (LEDs, switches, UART, OLED, etc.).

#### Address Decoding

In our `Wrapper.v`, we need to decide whether we want to **access** (can be "read" or "write") DMEM or MMIO peripherals. To do so, we introduce lots of `dec_*` signals to indicate whether we want to access DMEM or MMIO peripherals.

{% code lineNumbers="true" %}
```verilog
//----------------------------------------------------------------
// Address Decode signals
//---------------------------------------------------------------
// 'enable' signals from data memory address decoding
reg
    dec_DMEM,
    dec_UART_RX_VALID,
    dec_UART_RX,
    dec_UART_TX_READY,
    dec_UART_TX,
    dec_OLED_COL,
    dec_OLED_ROW,
    dec_OLED_DATA,
    dec_OLED_CTRL,
    dec_ACCEL_DATA,
    dec_ACCEL_DREADY,
    dec_LED,
    dec_DIP,
    dec_PB,
    dec_SEVENSEG,
    dec_CYCLECOUNT  // Cycle count signal
    ;

//----------------------------------------------------------------
// Data memory address decoding
//----------------------------------------------------------------
always @(*) begin
    dec_DMEM          <= 1'b0;
    bad_MEM_addr      <= 1'b0;
    dec_UART_RX_VALID <= 1'b0;
    dec_UART_RX       <= 1'b0;
    dec_UART_TX_READY <= 1'b0;
    dec_UART_TX       <= 1'b0;
    dec_OLED_COL      <= 1'b0;
    dec_OLED_ROW      <= 1'b0;
    dec_OLED_DATA     <= 1'b0;
    dec_OLED_CTRL     <= 1'b0;
    dec_ACCEL_DATA    <= 1'b0;
    dec_ACCEL_DREADY  <= 1'b0;
    dec_LED           <= 1'b0;
    dec_DIP           <= 1'b0;
    dec_PB            <= 1'b0;
    dec_SEVENSEG      <= 1'b0;
    dec_CYCLECOUNT    <= 1'b0;

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
end
```
{% endcode %}

#### Connection to RV

Different from the [Lec 03 microarchitecture](https://wenbo-notes.gitbook.io/ddca-notes/lec/lec-03-risc-v-isa-and-microarchitecture#support-for-lui-and-auipc), here we added the MMIO part inside.

<figure><img src="../.gitbook/assets/lab02-high-level-explanation.png" alt=""><figcaption></figcaption></figure>

{% stepper %}
{% step %}
#### Instruction fetch path

1. CPU outputs **PC**. This is done in `RV.v` as `PC` is an output from CPU.
2.  Wrapper uses PC to fetch the `Instr` from instruction memory.&#x20;

    {% code lineNumbers="true" %}
    ```verilog
    //----------------------------------------------------------------
    // IROM read
    //----------------------------------------------------------------
    always @(*) begin  // @posedge CLK only if using synch read for memory
        Instr = ( ( PC[31:IROM_DEPTH_BITS] == IROM_BASE[31:IROM_DEPTH_BITS]) && // To check if address is in the valid range
        (PC[1:0] == 2'b00) )? // and is word-aligned - we do not support instruction sizes other than 32.
        IROM[PC[IROM_DEPTH_BITS-1:2]] : 32'h00000013 ; // If the address is invalid, the instruction fetched is NOP. 
        // This can be changed to trigger an exception instead if need be.
    end
    ```
    {% endcode %}
3. Wrapper sends `Instr` back into CPU.  This done in `RV.v` as `Instr` is an input to CPU.
{% endstep %}

{% step %}
#### Data access path

The data access path has two parts, one is to load data from either DMEM or MMIO, the other is to store data to either DMEM or MMIO.

1. **Load**: Loading from both DMEM and MMIO peripherals look identical to CPU (in our implementation, we read both together first), only Wrapper decides the source.
   * CPU computes address in ALU -> `ALUResult`.
   * CPU asserts `MemRead = 1`. (`MemRead` is just `MemtoReg` in our microarchitecture)
   *   Wrapper checks `ALUResult` and enable corresponding `dec_*` signal to indicate which memory we want to access (read from here)

       {% code lineNumbers="true" %}
       ```verilog
       // Derive the decoded MMIO read signal
       assign dec_MMIO_read = MemRead || dec_DIP || dec_PB || dec_UART_RX_VALID || dec_UART_RX || dec_UART_TX_READY || dec_UART_TX || dec_CYCLECOUNT || dec_ACCEL_DATA || dec_ACCEL_DREADY ;
       ```
       {% endcode %}

       *   If address ∈ **DMEM range** (`dec_DMEM == 1`)-> access (read from) DMEM, and store the read data into `ReadData_DMEM`.

           {% code lineNumbers="true" %}
           ```verilog
           //----------------------------------------------------------------
           // Asych DMEM read - //Uncomment the following block (3 lines) if NOT using synch read for memory
           //----------------------------------------------------------------
           always @(*) begin
               ReadData_DMEM <= DMEM[ALUResult[DMEM_DEPTH_BITS-1:2]];  // async read
           end
           ```
           {% endcode %}
       *   If address ∈ **MMIO range** (`dec_MMIO_* == 1`) -> access (read from) the MMIO peripheral and store the read data into `ReadData_MMIO`.

           {% code lineNumbers="true" %}
           ```verilog
           //----------------------------------------------------------------
           // MMIO read
           //----------------------------------------------------------------
           always @(*) begin  // @posedge CLK only if using synch read for memory
               if (dec_DIP) ReadData_MMIO <= {{31 - N_DIPs + 1{1'b0}}, DIP};
               else if (dec_PB) ReadData_MMIO <= {{31 - N_PBs + 1{1'b0}}, PB};
               else if (dec_UART_RX && UART_RX_valid) ReadData_MMIO <= {24'd0, UART_RX};
               else if (dec_UART_RX_VALID) ReadData_MMIO <= {31'd0, UART_RX_valid};
               else if (dec_UART_TX_READY) ReadData_MMIO <= {31'd0, UART_TX_ready};
               else if (dec_CYCLECOUNT) ReadData_MMIO <= cycle_count;
               else if (dec_ACCEL_DATA) ReadData_MMIO <= ACCEL_Data;
               else  // dec_ACCEL_DREADY // the default else to avoid the statement from being incomplete
                   ReadData_MMIO <= {31'd0, ACCEL_DReady};
           end
           ```
           {% endcode %}
       *   After that, we delay the decoded signals

           {% code lineNumbers="true" %}
           ```verilog
           //----------------------------------------------------------------
           // Delaying the decoded signals for multiplexing (delay only if using synch read for memory)
           //----------------------------------------------------------------
           always @(*) begin  // @posedge CLK only if using synch read for memory
               dec_DMEM_W <= dec_DMEM;
               dec_MMIO_read_W <= dec_MMIO_read;
           end
           ```
           {% endcode %}
   *   Wrapper puts that value into `ReadData_in`. (This is where the I/O Multiplexing comes into play)

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
   * CPU reads `ReadData_in` and writes into register file.
2. **Store**: It will either update memory or trigger side effects on peripherals.
   * CPU computes address in ALU -> `ALUResult`.
   * CPU outputs the value we want to store in `WriteData_out`.
   *   CPU asserts `MemWrite_out = 1`. (This is the `{4{MemWrite}}`, where `MemWrite` is in our Lec 03 microarchitecture)

       ```verilog
       assign MemWrite = MemWrite_out[3] || MemWrite_out[2] || MemWrite_out[1] || MemWrite_out[0];
       ```
   * Wrapper checks `ALUResult`:
     *   If address ∈ **DMEM range** -> write `WriteData_out` into DMEM.

         {% code lineNumbers="true" %}
         ```verilog
         //----------------------------------------------------------------
         // DMEM write
         //----------------------------------------------------------------
         localparam NUM_COL = 4;
         localparam COL_WIDTH = 8;
         integer i;
         always @(posedge CLK) begin
             if (MemWrite) begin
                 for (i = 0; i < NUM_COL; i = i + 1) begin
                     if (MemWrite_out[i]) begin
                         if (dec_DMEM) begin
                             DMEM[ALUResult[DMEM_DEPTH_BITS-1:2]][i*COL_WIDTH +: COL_WIDTH] <= WriteData_out[i*COL_WIDTH +: COL_WIDTH];
                         end
                     end
                 end
             end
             //ReadData_DMEM <= DMEM[ALUResult[DMEM_DEPTH_BITS-1:2]] ; //Uncomment only if only using synch read for memory
         end
         ```
         {% endcode %}
     *   If address ∈ **MMIO range** -> forward `WriteData_out` into peripheral (e.g. set LEDs, update 7-seg).

         {% code lineNumbers="true" %}
         ```verilog
         //----------------------------------------------------------------
         // SevenSeg write
         //----------------------------------------------------------------
         integer j;
         always @(posedge CLK) begin
             if (MemWrite) begin
                 for (j = 0; j < NUM_COL; j = j + 1) begin
                     if (MemWrite_out[j]) begin
                         if (RESET) SEVENSEGHEX <= 32'd0;
                         else if (dec_SEVENSEG)
                             SEVENSEGHEX[j*COL_WIDTH+:COL_WIDTH] <= WriteData_out[j*COL_WIDTH+:COL_WIDTH];
                     end
                 end
             end
         end

         //----------------------------------------------------------------
         // Memory-mapped LED write
         //----------------------------------------------------------------
         always @(posedge CLK) begin
             if (RESET) LED_OUT <= 0;
             else if (MemWrite_out[0] && dec_LED) LED_OUT <= WriteData_out[N_LEDs_OUT-1 : 0];
         end
         ```
         {% endcode %}
   * CPU does not expect anything on `ReadData_in`. (But will it mess up?)
{% endstep %}
{% endstepper %}

### RISC-V processor

Our RISC-V processor, or simply denoted as RV processor, is designed in `RV.v`, which also has the following hierarchy

```bash
- RV.v                  // Our CPU!
    |
    - ALU.v
        |
        - Shifter.v
    - Decoder.v
    - Extend.v
    - PC_Logic.v
    - ProgramCounter.v
    - RegFile.v
```

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
