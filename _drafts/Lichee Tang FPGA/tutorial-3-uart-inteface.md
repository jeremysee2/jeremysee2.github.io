---
layout: post
title: 'Tutorial 3: UART Inteface'
menus: ''
date: 2021-03-31 16:00:00 +0000

---
### Tutorial 3: UART Interface

In this tutorial, we will create a UART interface to send and receive data with your computer. This introduces the concept of a First In First Out (FIFO) buffer between the external UART interface and the FPGA logic.

[Click here for the introduction to FPGAs on the Lichee Tang board](https://jeremysee2.github.io/2021/03/28/the-15-fpga-with-20-000-luts/).

[Click here for Tutorial 2, controlling a seven segment display](https://jeremysee2.github.io/2021/03/31/tutorial-2-seven-segment-display/).

Let's define some parameters for the UART interface we're using.

* 115200 baud rate
* 8 data bits
* No parity bit
* 1 stop bit
* No flow control

For a detailed explanation of UART, watch [this video by nandland](https://www.youtube.com/watch?v=Vh0KdoXaVgU&t=1172s). We'll focus on the implementation in Verilog, and how to use it with the Lichee Tang board.

#### UART Receiver

We use a state machine to perform a sequence of actions to wait for data, look for the start bit, look for data bits, look for the stop bit and clean up the state machine by going back to the `IDLE` state. For our `UART_RX` module, we'll define a `clk` input and a `RXserial` input, a `datavalid` output and a `RXbyte` bus output.

We'll also add in the states for our state machine, so we can reference them intuitively instead of relying on values like `3`b001`. We define internal signals as well.

`rClockCount` divides the clock so we only read the data in once, to avoid re-sampling the same data. `rBitIndex` keeps track of which data bit we are currently at, while `rRXByte` keeps track of the actual data. `oRXDataValid`is an output that we use to signal whether the data received is in the correct format, and will be useful for downstream modules that take in data from this UART module. `rSMMain` is the main variable that stores our states for this state machine.

Note that the `case` statement is used for the state machines, rather than daisy-chained `if` `else` blocks.

```verilog
module UART_RX (
   input        i_Clock,
   input        i_RX_Serial,
   output       o_RX_Data_Valid,
   output [7:0] o_RX_Byte
   );
  
  // States for finite state machine (FSM)
  parameter IDLE         = 3'b000;
  parameter RX_START_BIT = 3'b001;
  parameter RX_DATA_BITS = 3'b010;
  parameter RX_STOP_BIT  = 3'b011;
  parameter CLEANUP      = 3'b100;
  
  // Internal signals to count clock, keep track of bit position
  reg [7:0]     r_Clock_Count   = 0;
  reg [2:0]     r_Bit_Index     = 0; //8 bits total
  reg [7:0]     r_RX_Byte       = 0;
  reg           o_RX_Data_Valid = 0;
  reg [2:0]     r_SM_Main       = 0;

endmodule
```

s