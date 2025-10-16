# Lab Demo Resources

Starting from the Lab 02 (including Lab 02), you will be required to use one assembly program for the hardware implementation demo. Based on our experience, we think [**UART**](https://wenbo-notes.gitbook.io/cg2111a-notes/studio/studio-9-serial-communication#u-s-art-communication) is the best practice for the demo. As up until Lab 03, all we need to do is to input numbers, and the CPU processes the numbers and show the output on the SevenSeg. To achieve that easier, we provide our **UART** example program for inputing 32-bit numbers into our processor using RealTerm and process them. (Different from Prof. Rajesh's one, but more practical for Lab Demo usage)

{% code lineNumbers="true" %}
```armasm
# CG3207 Lab 3 UART Based Arithmetic emulator
# ----------------------------------------------------------------
# Supported commands:
#   'a' -> ADD (t4 = t2 + t3)
#   's' -> SUB (t4 = t2 - t3)
#   'm' -> MUL (t4 = t2 * t3)
#   'd' -> DIVU (t4 = t2 / t3)
#   Sent as 4 raw bytes via RealTerm (MSB first)
#
# Registers:
#   s0?s5 : UART MMIO base & offsets
#   s7    : bitmask 0x1
#   t0    : flag/temp
#   t1    : command (ASCII)
#   t2    : operand1 (32-bit)
#   t3    : operand2 (32-bit)
#   t4    : result (32-bit)
#   t5    : byte counter
#   t6    : current received byte
# ----------------------------------------------------------------

.eqv MMIO_BASE          0xFFFF0000
.eqv UART_RX_VALID_OFF  0x00  # RX valid flag (bit0=1 means data ready)
.eqv UART_RX_OFF        0x04  # RX data register
.eqv UART_TX_READY_OFF  0x08  # TX ready flag (bit0=1 means ready)
.eqv UART_TX_OFF        0x0C  # TX data register
.eqv SEVENSEG_OFF       0x80  # 7-segment/LED output
.eqv LSB_MASK           0x01  # mask for bit0

.text
main:
    # Initialization of MMIO addresses
    li   s0, MMIO_BASE
    addi s1, s0, UART_RX_VALID_OFF
    addi s2, s0, UART_RX_OFF
    addi s3, s0, UART_TX_READY_OFF
    addi s4, s0, UART_TX_OFF
    addi s5, s0, SEVENSEG_OFF
    li   s7, LSB_MASK

# Main processing loop
loop:
# Read command byte
READ_CMD:
    lw   t0, 0(s1)          # t0 = UART_RX_VALID
    and  t0, t0, s7         # check bit0
    beqz t0, READ_CMD       # loop until data ready
    lw   t1, 0(s2)          # t1 = received command (ASCII)
    andi t1, t1, 0xFF       # keep only low 8 bits

# Echo command back to RealTerm for confirmation
ECHO_CMD:
    lw   t0, 0(s3)          # check TX_READY
    and  t0, t0, s7
    beqz t0, ECHO_CMD       # wait until ready
    sw   t1, 0(s4)          # transmit command back
    

# Read Operand 1 (4 bytes or 32 bits, MSB first)
    li   t5, 4              # t5 = byte counter (4)
    li   t2, 0              # t2 = operand1

READ_OP1_LOOP:
    lw   t0, 0(s1)          # check RX_VALID
    and  t0, t0, s7
    beqz t0, READ_OP1_LOOP  # wait for next byte
    lw   t6, 0(s2)          # t6 = received byte
    andi t6, t6, 0xFF
    # echo the received byte back to realterm
ECHO_OP1_BYTE:
    lw   t0, 0(s3)
    and  t0, t0, s7
    beqz t0, ECHO_OP1_BYTE
    sw   t6, 0(s4)
    # Build the 32-bit number
    slli t2, t2, 8           # shift previous bytes left
    or   t2, t2, t6          # append new byte
    addi t5, t5, -1
    bnez t5, READ_OP1_LOOP  # repeat 4 times

# After loop:
#   t2 = operand1 (32-bit)

# Read Operand 2 (4 bytes or 32 bits, MSB first)
    li   t5, 4              # t5 = byte counter (4)
    li   t3, 0              # t3 = operand2

READ_OP2_LOOP:
    lw   t0, 0(s1)
    and  t0, t0, s7
    beqz t0, READ_OP2_LOOP
    lw   t6, 0(s2)
    andi t6, t6, 0xFF
    # Echo the received byte back
ECHO_OP2_BYTE:
    lw   t0, 0(s3)
    and  t0, t0, s7
    beqz t0, ECHO_OP2_BYTE
    sw   t6, 0(s4)
    # Build the 32-bit number
    slli t3, t3, 8
    or   t3, t3, t6
    addi t5, t5, -1
    bnez t5, READ_OP2_LOOP

# After loop:
#   t3 = operand2 (32-bit)

# Decode and execute command
CMD_CHECK:
    li   t0, 'a'
    beq  t1, t0, DO_ADD
    li   t0, 's'
    beq  t1, t0, DO_SUB
    li   t0, 'm'
    beq  t1, t0, DO_MUL
    li   t0, 'd'
    beq  t1, t0, DO_DIVU
    j    loop                # invalid command, restart

# ADD: t4 = t2 + t3
DO_ADD:
    add  t4, t2, t3
    sw   t4, 0(s5)           # show result on 7-seg
    j    loop

# SUB: t4 = t2 - t3
DO_SUB:
    sub  t4, t2, t3
    sw   t4, 0(s5)
    j    loop

# MUL: t4 = t2 * t3 (32-bit)
DO_MUL:
    mul  t4, t2, t3
    sw   t4, 0(s5)
    j    loop

# DIVU: t4 = t2 / t3 (unsigned)
DO_DIVU:
    beqz t3, DIV_ZERO        # protect div-by-zero
    divu t4, t2, t3
    sw   t4, 0(s5)
    j    loop

# Handle divide-by-zero
DIV_ZERO:
    li   t4, 0xFFFFFFFF
    sw   t4, 0(s5)
    j    loop
```
{% endcode %}

The lab manual has provided a detailed [guide](https://nus-cg3207.github.io/labs/rv_resources/peripherals/#uart) to setup the RealTerm.
