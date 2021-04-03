---
layout: post
title: The $15 FPGA with 20,000 LUTs
menus: ''
date: 

---
## Lichee Tang: The cheapest beginner FPGA

Have you ever wanted to start learning FPGAs but just can't spare the $80-$150 for an official Xilinx/Altera board from places like Digilent? Let me introduce you to what is possibly the best beginner FPGA for learning RTL (Verilog/VHDL)!

![](/uploads/tang_fpga.jpg)

The Lichee Tang Primer is a low-cost FPGA board made by Sipeed, using Anlogic's EG4S2BG256 FPGA. It's a great value for money, with 20,000 LUT4 logic elements and an onboard JTAG interface for uploading your bitstreams directly to the FPGA or SPI flash. In fact, it's vendor IDE is especially user-friendly, being able to synthesise bitstreams in a matter of seconds, instead of minutes like Quartus/Vivado from the major players.

So, what are the downsides? Firstly, it's just not well supported by the community or industry - meaning you'll need to learn those pesky Quartus/Vivado tools eventually if you go into industry. Secondly, you'll need to dig through documentation to use specific features (SERDES, ADC, PLL...) and vendor-provided IP.

However, for a beginner like me, we can wait to sort out those problems later... I just want to have a cheap and user-friendly platform to learn Verilog! 

### Toolchain Setup

To set up the toolchain for this board, you can follow the official tutorial at the [Sipeed wiki](https://tang.sipeed.com/en/getting-started/). I'll briefly go through the setup steps:

1. Download the appropriate copy of [Tang Dynasty IDE from Sipeed](http://dl.sipeed.com/TANG/Primer/IDE)
2. Download the datasheet for the board and IDE from [here](https://github.com/kprasadvnsi/Anlogic_Doc_English)
3. For Linux, follow the setup guidelines [here](http://cgoxopx.sinriv.com/psg/2019-8-31:19:28:32) and run the `td -gui` command to open the IDE
4. For Windows, install using the executable and [set your system time](https://runasdate.en.softonic.com/) to be before 2018, to enable the provided Sipeed license. Then, you will be able to run the TD IDE with the ability to synthesise the bitstream
5. Install the USB drivers [here](https://tang.sipeed.com/en/getting-started/installing-usb-driver/) 
6. Double-check that your setup is valid by [running the Blinky example](https://tang.sipeed.com/en/getting-started/getting-to-blinky/)

### Tutorial 0: Logic Gates

To illustrate the differences between an FPGA and an embedded microcontroller, it's good practice to start by coding some logic gates in the FPGA to link some buttons and LEDs.

### Tutorial 1: Seven Segment Display

In this tutorial, we will control a seven segment display using the FPGA. This will introduce concepts such as `module` where code can be written and reused, a similar paradigm to Object Oriented Programming.

### Tutorial 2: UART Interface

### Tutorial 3: VGA Interface

### References

For this tutorial I referenced the following sources:

* [Nandland tutorials](https://github.com/nandland/nandland)
* [Seven segment tutorial](https://github.com/ombhilare999/Seven-Segment-with-Tang-Primer-FPGA)
* [VGA tutorial](https://github.com/piotr-go/Lichee-Tang)
* [Official Sipeed tutorial](https://github.com/Lichee-Pi/Tang_FPGA_Examples)
* [Picorv32 tutorial](https://github.com/nekomona/picorv32-tang)