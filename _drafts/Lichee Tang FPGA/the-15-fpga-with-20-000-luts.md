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