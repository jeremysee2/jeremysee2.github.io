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

Now, let's add in the logic for the state machine itself. The entire state machine is nested within an `always` block, sensitive to `posedge i_Clk`. 

The `IDLE` state resets internal signals, and looks for the start bit of `1`b0`. If it's detected, it goes to the next state.

`RX-START-BIT` uses the internal clock counter to divide the 24MHz clock to 115200, matching the baud rate of the UART line. It uses that to double-check that the start bit has been set, then sets the `RX-DATA-BITS` state. Else, it goes back to the `IDLE` state.

`RX-DATA-BITS` waits `CLKS-PER-BIT -1` clock cycles to sample the incoming data, using the `r-Clock-Count` variable. Once done waiting, in the `else` block, it samples the data bit into the correct position of the `r-RX-Byte[r-Bit-Index]` for storage, until all 8 bits are received. Once the entire byte has been received, it goes to the next state to look for the stop bit.

`RX-STOP-BIT` waits `CLKS-PER-BIT -1` clock cycles, assuming that the stop bit will appear and pass. Then, it sends the state machine to the `CLEANUP` state, and raises high the `o-RX-Data-Valid` signal, to indicate to downstream modules that a data byte is valid, and ready to be read.

`CLEANUP` is the final state, which adds a one clock delay for downstream modules to read the data byte. It then sets the `o-RX-Data-Valid` signal low to indicate that the output data is invalid. Downstream modules can use this signal to ensure they do not read the same output data twice.

Finally, there are some assign statements to tie internal signals to output ports.

```verilog
/////////////////////////////////////////////////////////////////////
// File Downloaded from http://www.nandland.com
/////////////////////////////////////////////////////////////////////
// This file contains the UART Receiver.  This receiver is able to
// receive 8 bits of serial data, one start bit, one stop bit,
// and no parity bit.  When receive is complete o_rx_dv will be
// driven high for one clock cycle.
// 
// Set Parameter CLKS_PER_BIT as follows:
// CLKS_PER_BIT = (Frequency of i_Clock)/(Frequency of UART)
// Example: 24 MHz Clock, 115200 baud UART
// (24000000)/(115200) = 208
 
module UART_RX
  #(parameter CLKS_PER_BIT = 208)
  (
   input        i_Clock,
   input        i_RX_Serial,
   output       o_RX_Data_Valid,
   output [7:0] o_RX_Byte
   );
   
  parameter IDLE         = 3'b000;
  parameter RX_START_BIT = 3'b001;
  parameter RX_DATA_BITS = 3'b010;
  parameter RX_STOP_BIT  = 3'b011;
  parameter CLEANUP      = 3'b100;
  
  reg [7:0]     r_Clock_Count   = 0;
  reg [2:0]     r_Bit_Index     = 0; //8 bits total
  reg [7:0]     r_RX_Byte       = 0;
  reg           o_RX_Data_Valid = 0;
  reg [2:0]     r_SM_Main       = 0;
  
  
  // Purpose: Control RX state machine
  always @(posedge i_Clock)
  begin
      
    case (r_SM_Main)
      IDLE :
        begin
          o_RX_Data_Valid   <= 1'b0;
          r_Clock_Count     <= 0;
          r_Bit_Index       <= 0;
          
          if (i_RX_Serial == 1'b0)          // Start bit detected
            r_SM_Main <= RX_START_BIT;
          else
            r_SM_Main <= IDLE;
        end
      
      // Check middle of start bit to make sure it's still low
      RX_START_BIT :
        begin
          if (r_Clock_Count == (CLKS_PER_BIT-1)/2)
          begin
            if (i_RX_Serial == 1'b0)
            begin
              r_Clock_Count <= 0;  // reset counter, found the middle
              r_SM_Main     <= RX_DATA_BITS;
            end
            else
              r_SM_Main <= IDLE;
          end
          else
          begin
            r_Clock_Count <= r_Clock_Count + 1;
            r_SM_Main     <= RX_START_BIT;
          end
        end // case: RX_START_BIT
      
      
      // Wait CLKS_PER_BIT-1 clock cycles to sample serial data
      RX_DATA_BITS :
        begin
          if (r_Clock_Count < CLKS_PER_BIT-1)
          begin
            r_Clock_Count <= r_Clock_Count + 1;
            r_SM_Main     <= RX_DATA_BITS;
          end
          else
          begin
            r_Clock_Count          <= 0;
            r_RX_Byte[r_Bit_Index] <= i_RX_Serial;
            
            // Check if we have received all bits
            if (r_Bit_Index < 7)
            begin
              r_Bit_Index <= r_Bit_Index + 1;
              r_SM_Main   <= RX_DATA_BITS;
            end
            else
            begin
              r_Bit_Index <= 0;
              r_SM_Main   <= RX_STOP_BIT;
            end
          end
        end // case: RX_DATA_BITS
      
      
      // Receive Stop bit.  Stop bit = 1
      RX_STOP_BIT :
        begin
          // Wait CLKS_PER_BIT-1 clock cycles for Stop bit to finish
          if (r_Clock_Count < CLKS_PER_BIT-1)
          begin
            r_Clock_Count <= r_Clock_Count + 1;
     	    r_SM_Main     <= RX_STOP_BIT;
          end
          else
          begin
       	    o_RX_Data_Valid <= 1'b1;
            r_Clock_Count   <= 0;
            r_SM_Main       <= CLEANUP;
          end
        end // case: RX_STOP_BIT
      
      
      // Stay here 1 clock
      CLEANUP :
        begin
          r_SM_Main         <= IDLE;
          o_RX_Data_Valid   <= 1'b0;
        end
      
      
      default :
        r_SM_Main <= IDLE;
      
    endcase
  end    
  
  assign o_RX_DV   = o_RX_Data_Valid;
  assign o_RX_Byte = r_RX_Byte;
  
endmodule // UART_RX
```

Now, let's write the testbench to ensure that our state machine is able to read and output data correctly.

```verilog
//////////////////////////////////////////////////////////////////////
// File Downloaded from http://www.nandland.com
//////////////////////////////////////////////////////////////////////

// This testbench will exercise the UART RX.
// It sends out byte 0x37, and ensures the RX receives it correctly.
`timescale 1ns/10ps
`include "UART_RX.v"

module UART_RX_tb();

  // Testbench uses a 24 MHz clock (same as Lichee Tang board)
  // Want to interface to 115200 baud UART
  // 24000000 / 115200 = 208 Clocks Per Bit.
  parameter c_CLOCK_PERIOD_NS = 41;
  parameter c_CLKS_PER_BIT    = 208;
  parameter c_BIT_PERIOD      = 8600;
  
  reg r_Clock = 0;
  reg r_RX_Serial = 1;
  wire [7:0] w_RX_Byte;
  

  // Takes in input byte and serializes it 
  task UART_WRITE_BYTE;
    input [7:0] i_Data;
    integer     ii;
    begin
      
      // Send Start Bit
      r_RX_Serial <= 1'b0;
      #(c_BIT_PERIOD);
      #1000;
      
      // Send Data Byte
      for (ii=0; ii<8; ii=ii+1)
        begin
          r_RX_Serial <= i_Data[ii];
          #(c_BIT_PERIOD);
        end
      
      // Send Stop Bit
      r_RX_Serial <= 1'b1;
      #(c_BIT_PERIOD);
     end
  endtask // UART_WRITE_BYTE
  
  
  UART_RX #(.CLKS_PER_BIT(c_CLKS_PER_BIT)) UART_RX_INST
    (.i_Clock(r_Clock),
     .i_RX_Serial(r_RX_Serial),
     .o_RX_Data_Valid(),
     .o_RX_Byte(w_RX_Byte)
     );
  
  always
    #(c_CLOCK_PERIOD_NS/2) r_Clock <= !r_Clock;

  
  // Main Testing:
  initial
    begin
      // Send a command to the UART (exercise Rx)
      @(posedge r_Clock);
      UART_WRITE_BYTE(8'h37);
      @(posedge r_Clock);
            
      // Check that the correct command was received
      if (w_RX_Byte == 8'h37)
        $display("Test Passed - Correct Byte Received");
      else
        $display("Test Failed - Incorrect Byte Received");
    $finish();
    end
  
  initial 
  begin
    // Required to dump signals to EPWave
    $dumpfile("dump.vcd");
    $dumpvars(0);
  end
  
endmodule
```

s