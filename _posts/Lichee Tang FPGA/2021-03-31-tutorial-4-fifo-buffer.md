---
layout: post
title: 'Tutorial 4: FIFO Buffer'
menus: ''
date: 2021-03-31 16:00:00 +0000

---
### FIFO Buffer

In our [previous tutorial](https://jeremysee2.github.io/2021/03/31/tutorial-3-uart-inteface/), our UART interface could only send and receive one byte at a time. To solve that problem, let's implement a First In First Out (FIFO) buffer to hold the previous two values in data registers. We define some specifications for our FIFO buffer below.

* 16-bit data bus
* Duplex read/write
* Read and write enable
* Full and Empty flags
* Overflow and underflow flags

We start by defining the ports to our module.

```verilog
module fifo_memory (
    input i_Clock,
    input i_Reset,
    input i_Write_En,
    input i_Read_En,
    input  [c_WIDTH:0] i_Data_In,
    output [c_WIDTH:0] o_Data_Out,
    output reg fifo_full,
    output reg fifo_empty,
    output reg fifo_overflow,
    output reg fifo_underflow
    );
endmodule
```

Then, we define the internal signals we use to store and access the memory.

```verilog
    // Internal memory, 7 16-bit wide registers
    parameter c_DEPTH = 7;
    parameter c_WIDTH = 15;
    reg [c_WIDTH:0] memory [0:c_DEPTH];
    reg [c_DEPTH:0]  wraddr = 0;
    reg [c_DEPTH:0]  rdaddr = 0;
    reg [c_WIDTH:0] r_Data_Out;
```

After, we add in a reset block to initialise our outputs and internal signals.

```verilog
    // Reset block
    always @(posedge i_Clock, negedge i_Reset) begin
        if (!i_Reset) begin
            // Reset output flags
            {fifo_full, 
             fifo_empty,
             fifo_overflow,
             fifo_underflow} <= 4'b0000;
            
            // Reset internal signals
            wraddr <= 0;
            rdaddr <= 0;
            r_Data_Out <= 0;
        end
    end
```

Then, we define logic for reading from and writing to the internal memory of the FIFO buffer, sequentially.

```verilog
    // Writing to FIFO
    always @(posedge i_Clock) begin
        if (i_Write_En) begin
            memory[wraddr] <= i_Data_In;

            // Incrementing wraddr pointer
            if ((!fifo_full) || (i_Read_En)) begin
                wraddr <= wraddr + 1'b1;
                fifo_overflow <= 1'b0;
            end
            else
                fifo_overflow <= 1'b1;
        end
    end

    // Reading from FIFO
    always @(posedge i_Clock) begin
        if (i_Read_En) begin
            r_Data_Out <= memory[rdaddr];

            // Incrementing raddr pointer
            if (!fifo_empty) begin
                rdaddr <= rdaddr + 1'b1;
                fifo_underflow <= 1'b0;
            end
            else
                fifo_underflow <= 1'b1;
        end
    end

    assign o_Data_Out = r_Data_Out;
```

Next, we want to manage the `fifo-full` and `fifo-empty` flags that we use to guide read and write operations. This section is referenced from [zipcpu](https://zipcpu.com/blog/2017/07/29/fifo.html), as it provides an efficient way to set read and write flags in one clock cycle.

```verilog

    // Calculating full/empty flags, referenced from zipcpu.com
    wire	[c_DEPTH:0]	dblnext, nxtread;
    assign	dblnext = wraddr + 2;
    assign	nxtread = rdaddr + 1'b1;

    always @(posedge i_Clock)
        if (!i_Reset)
        begin
            fifo_full <= 1'b0;
            fifo_empty <= 1'b1;
        end else casez({ i_Write_En, i_Read_En, !fifo_full, !fifo_empty })
        4'b01?1: begin	// A successful read
            fifo_full  <= 1'b0;
            fifo_empty <= (nxtread == wraddr);
        end
        4'b101?: begin	// A successful write
            fifo_full <= (dblnext == rdaddr);
            fifo_empty <= 1'b0;
        end
        4'b11?0: begin	// Successful write, failed read
            fifo_full  <= 1'b0;
            fifo_empty <= 1'b0;
        end
        4'b11?1: begin	// Successful read and write
            fifo_full  <= fifo_full;
            fifo_empty <= 1'b0;
        end
        default: begin end
        endcase
```

Lastly, we bring it all together for the final file `fifo_memory.v`.

```verilog
module fifo_memory (
    input i_Clock,
    input i_Reset,
    input i_Write_En,
    input i_Read_En,
    input  [c_WIDTH:0] i_Data_In,
    output [c_WIDTH:0] o_Data_Out,
    output reg fifo_full,
    output reg fifo_empty,
    output reg fifo_overflow,
    output reg fifo_underflow
    );

    // Internal memory, 7 16-bit wide registers
    parameter c_DEPTH = 7;
    parameter c_WIDTH = 15;
    reg [c_WIDTH:0] memory [0:c_DEPTH];
    reg [c_DEPTH:0]  wraddr = 0;
    reg [c_DEPTH:0]  rdaddr = 0;
    reg [c_WIDTH:0] r_Data_Out;

    // Reset block
    always @(posedge i_Clock, negedge i_Reset) begin
        if (!i_Reset) begin
            // Reset output flags
            {fifo_full, 
             fifo_empty,
             fifo_overflow,
             fifo_underflow} <= 4'b0000;
            
            // Reset internal signals
            wraddr <= 0;
            rdaddr <= 0;
            r_Data_Out <= 0;
        end
    end

    // Writing to FIFO
    always @(posedge i_Clock) begin
        if (i_Write_En) begin
            memory[wraddr] <= i_Data_In;

            // Incrementing wraddr pointer
            if ((!fifo_full) || (i_Read_En)) begin
                wraddr <= wraddr + 1'b1;
                fifo_overflow <= 1'b0;
            end
            else
                fifo_overflow <= 1'b1;
        end
    end

    // Reading from FIFO
    always @(posedge i_Clock) begin
        if (i_Read_En) begin
            r_Data_Out <= memory[rdaddr];

            // Incrementing raddr pointer
            if (!fifo_empty) begin
                rdaddr <= rdaddr + 1'b1;
                fifo_underflow <= 1'b0;
            end
            else
                fifo_underflow <= 1'b1;
        end
    end

    assign o_Data_Out = r_Data_Out;

    // Calculating full/empty flags, referenced from zipcpu.com
    wire	[c_DEPTH:0]	dblnext, nxtread;
    assign	dblnext = wraddr + 2;
    assign	nxtread = rdaddr + 1'b1;

    always @(posedge i_Clock)
        if (!i_Reset)
        begin
            fifo_full <= 1'b0;
            fifo_empty <= 1'b1;
        end else casez({ i_Write_En, i_Read_En, !fifo_full, !fifo_empty })
        4'b01?1: begin	// A successful read
            fifo_full  <= 1'b0;
            fifo_empty <= (nxtread == wraddr);
        end
        4'b101?: begin	// A successful write
            fifo_full <= (dblnext == rdaddr);
            fifo_empty <= 1'b0;
        end
        4'b11?0: begin	// Successful write, failed read
            fifo_full  <= 1'b0;
            fifo_empty <= 1'b0;
        end
        4'b11?1: begin	// Successful read and write
            fifo_full  <= fifo_full;
            fifo_empty <= 1'b0;
        end
        default: begin end
        endcase
    
endmodule
```

Let's write a testbench to validate the output signals of our module. By now you should be familiar with the general structure of a testbench.

1. Describe test signals
2. Instantiate unit under test (can be multiple of them)
3. Put testbench logic under `initial` block to run once. Use `always` block for repeating logic
4. Use `if` statements to validate outputs and print outputs using `$display()`
5. Save output waveform using `$dumpfile()` and `$dumpvars()`

```verilog
`timescale 1ns/1ns
`include "fifo_memory.v"

module fifo_memory_tb ();
    
    // Test signals
    reg r_Clock = 0;
    reg r_Reset = 1;
    reg r_Write_En = 0;
    reg r_Read_En = 0;
    reg  [15:0] r_Data_In = 0;
    wire [15:0] w_Data_Out;
    wire w_fifo_full;
    wire w_fifo_empty;
    wire w_fifo_overflow;
    wire w_fifo_underflow;

    parameter c_CLOCK_PERIOD_NS = 10;


    // Instantiate module
    fifo_memory #(
        .c_DEPTH(7),
        .c_WIDTH(15)
    ) UUT (
        .i_Clock(r_Clock),
        .i_Reset(r_Reset),
        .i_Write_En(r_Write_En),
        .i_Read_En(r_Read_En),
        .i_Data_In(r_Data_In),
        .o_Data_Out(w_Data_Out),
        .fifo_full(w_fifo_full),
        .fifo_empty(w_fifo_empty),
        .fifo_overflow(w_fifo_overflow),
        .fifo_underflow(w_fifo_underflow)
        );

    // Testbench logic
    always
        #(c_CLOCK_PERIOD_NS/2) r_Clock <= !r_Clock;

    // Main Testing:
    initial
    begin
        // Initialise module through reset
        r_Reset = ~r_Reset;
        #10
        r_Reset = ~r_Reset;
        #10

        // Write two bytes
        r_Data_In  <= 16'hBEEF;
        r_Write_En <= 1'b1;
        #10;
        r_Write_En <= 1'b0;
        r_Read_En  <= 1'b1;
        #10
        // Check that the correct data was received
        if (w_Data_Out == 16'hBEEF)
        $display("Test Passed - Correct two bytes received");
        else
        $display("Test Failed - Incorrect two bytes received");
        

        // Try overflowing it
        r_Write_En <= 1'b0;
        r_Read_En  <= 1'b0;

        for (integer i = 16'h0; i < 16'h1FF; i = i + 1'b1) begin
            r_Data_In  <= i;
            r_Write_En <= 1'b1;
            #10;
        end
        r_Write_En <= 1'b0;
        r_Read_En  <= 1'b0;
        if (w_fifo_overflow)
        $display("Test Passed - Overflow flag works");
        else
        $display("Test Failed - Overflow flag failed");


        // Try underflowing it
        r_Write_En <= 1'b0;
        r_Read_En  <= 1'b0;

        for (integer i = 16'h0; i < 16'h2FF; i = i + 1'b1) begin
            r_Read_En <= 1'b1;
            #10;
        end
        r_Write_En <= 1'b0;
        r_Read_En  <= 1'b0;
        if (w_fifo_underflow)
        $display("Test Passed - Underflow flag works");
        else
        $display("Test Failed - Underflow flag failed");
        $finish();
    end

    initial 
    begin
    // Required to dump signals
    $dumpfile("dump.vcd");
    $dumpvars(0);
    end

endmodule
```

Running the simulation in `iverilog` and viewing in `gtkwave` gives the following result.

![](/uploads/fifo-gtkwave.PNG)

Congratulations! You've defined your first FIFO buffer. In practice, FIFO buffers are very useful for the following situations.

* Crossing clock domains
* Buffering high speed, infrequent data
* Aligning data for math operations
* Buffering data coming from software, to be sent out of the chip

Now, let's use this for our UART peripheral we designed in [Tutorial 3](https://jeremysee2.github.io/2021/03/31/tutorial-3-uart-inteface/)!

#### Complete UART Transceiver

Let's draw up a block diagram of what our UART transceiver should look like.

![](/uploads/uart-bd.png)

We'll connect the RX, TX and FIFO modules that we've already created in this fashion. For some simple processing, we'll include a module that converts lower case characters to upper case characters, using a simple arithmetic operation (`-32` to convert from lower to upper case).