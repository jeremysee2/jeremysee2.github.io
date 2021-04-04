---
layout: post
title: The $15 FPGA with 20,000 LUTs
date: 
menus: ''

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

### Tutorial 1: FPGA Basics

As this series of tutorials were inspired by [Nandland](https://www.youtube.com/channel/UCsdA-aNqtMA1_2T15aXePWw), I highly recommend you check out his videos before moving to the next few ones that actually involve implementing your designs on a physical FPGA.

For this tutorial, we will follow along with the [first lecture](https://www.youtube.com/watch?v=l_eo21vHxw0&list=PLnAoag7Ew-vr1M98Q5K2kLHxFQ5l0DU3B&index=4) of Nandland. Here, we will be setting up our development environment and writing a simple Verilog module and testbench with `iverilog` and `gtkwave`.

Firstly, we'll want to install `iverilog` and `gtkwave`. `iverilog` is the Verilog compiler to perform simulations, and `gtkwave`allows you to view the resulting waveform.

#### Installing `iverilog` and `gtkwave`

For Windows, download the setup executable [here](http://bleyer.org/icarus/). Run the installer and check the "Add to PATH" option to automatically add it to PATH, allowing you to call it from the terminal. This executable also allows you to install `gtkwave` at the same time. 

For Linux, you can install from premade packages [here](https://iverilog.fandom.com/wiki/Installation_Guide#Installation_From_Premade_Packages). Follow the instructions for your distro. For Ubuntu, add the Universe repository to your `/etc/apt/sources.list` and run the command `sudo apt-get install iverilog gtkwave`. 

#### Module Structure

In Verilog, a module is defined with the keyword `module`. The following is an example of how a module is defined.

    verilog
    module Switches_To_LEDs
      (input i_Switch_1,  
       input i_Switch_2,
       input i_Switch_3,
       input i_Switch_4,
       output o_LED_1,
       output o_LED_2,
       output o_LED_3,
       output o_LED_4);
           
    assign o_LED_1 = i_Switch_1;
    assign o_LED_2 = i_Switch_2;
    assign o_LED_3 = i_Switch_3;
    assign o_LED_4 = i_Switch_4;
     
    endmodule

#### Logic Gates

### Tutorial 2: Seven Segment Display

In this tutorial, we will control a seven segment display using the FPGA. This will introduce concepts such as `module` where code can be written and reused, a similar paradigm to Object Oriented Programming.

### Tutorial 3: UART Interface

### Tutorial 4: VGA Interface

### References

For this tutorial I referenced the following sources:

* [Nandland tutorials](https://github.com/nandland/nandland)
* [Seven segment tutorial](https://github.com/ombhilare999/Seven-Segment-with-Tang-Primer-FPGA)
* [VGA tutorial](https://github.com/piotr-go/Lichee-Tang)
* [Official Sipeed tutorial](https://github.com/Lichee-Pi/Tang_FPGA_Examples)
* [Picorv32 tutorial](https://github.com/nekomona/picorv32-tang)