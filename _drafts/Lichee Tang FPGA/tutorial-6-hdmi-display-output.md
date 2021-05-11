---
layout: post
title: 'Tutorial 6: HDMI Display Output'
date: 2021-04-02 16:00:00 +0000

---
### DVI/HDMI Display Output

In the previous tutorial, we covered the VGA display interface. It converts parallel RGB data into the analog VGA interface. Now, let's take a look at a modern video data protocol, HDMI.

HDMI is based on the DVI standard before it, which comprises several signals that carry both device description data, audio data and video data. Due to licensing issues, we will be implementing the older (and simpler) DVI standard, which does not include audio. Device data travels via I2C or CEC. Video data travels on the Transition Minimised Differential Signalling (TMDS) physical layer.

#### TMDS Signalling

This signalling standard encodes 8 bits of each colour (RGB) into 10 bits. The information is then transmitted at 10x the speed of the pixel clock. This format is called `8b/10b`. 

![](/uploads/dvi-specification.png)

TMDS comprises two different encoding schemes depending on whether pixel data or control data is being transmitted. This is similar to VGA, where we send control signals (HSYNC, VSYNC) in the blanking area of each frame. The pixel clock specification is calculated on the colour bit depth and resolution (how much data you need to send). As a general rule of thumb, this hasn't changed since our last encounter with VGA, except now your TMDS clock will be 10x the speed of your base pixel clock.

![](/uploads/display-timings.png)

##### Control Tokens

There are 4 10-bit control tokens used to transmit two bits of data. These are mapped to the HSYNC and VSYNC video signals, similar to VGA, and for synchronisation purposes. This is done in the blanking period. The tokens are:

#### References

* [fpga4fun: TMDS encoder Verilog code](https://www.fpga4fun.com/HDMI.html)
* [Understanding HDMI](https://docs.google.com/document/d/1v7AJK4cVG3uDJo_rn0X9vxMvBwXKBSL1VaJgiXgFo5A)
* [ProjectF: HDMI Timing](https://projectf.io/posts/video-timings-vga-720p-1080p/)
* [Xilinx XAPP460: TMDS I/O in Spartan 3](https://www.xilinx.com/support/documentation/application_notes/xapp460.pdf)