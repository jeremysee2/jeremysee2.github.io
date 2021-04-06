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

We use a state machine to perform a sequence of actions, from `IDLE` to `READY` to receive data and back to `IDLE` again when ready to receive data.