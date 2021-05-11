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

There are 4 10-bit control tokens used to transmit two bits of data. These are mapped to the HSYNC and VSYNC video signals, similar to VGA, and for synchronisation purposes. This is done in the blanking period. `c0` and `c1` represent the HSYNC and VSYNC signals respectively.

![](/uploads/xapp460-dvi-encoder.PNG)

The tokens are:

| c0 | c1 | bits |
| --- | --- | --- |
| 0 | 0 | 10'b0010101011 |
| 0 | 1 | 10'b0010101010 |
| 1 | 0 | 10'b1101010100 |
| 1 | 1 | 10'b1101010101 |

##### Data Island (HDMI Only)

Additionally, HDMI defines a Data Island to transmit audio data and auxiliary data. This includes InfoFrames and other descriptive data, making it the key difference between the two protocols. This uses another encoding scheme called `TERC4` which allows 4 bits of data per channel to be sent.

![](/uploads/xapp460-hdmi-encoder.PNG)

```verilog
case (D3, D2, D1, D0):
    0000: q_out[9:0] = 0b1010011100;
    0001: q_out[9:0] = 0b1001100011;
    0010: q_out[9:0] = 0b1011100100;
    0011: q_out[9:0] = 0b1011100010;
    0100: q_out[9:0] = 0b0101110001;
    0101: q_out[9:0] = 0b0100011110;
    0110: q_out[9:0] = 0b0110001110;
    0111: q_out[9:0] = 0b0100111100;
    1000: q_out[9:0] = 0b1011001100;
    1001: q_out[9:0] = 0b0100111001;
    1010: q_out[9:0] = 0b0110011100;
    1011: q_out[9:0] = 0b1011000110;
    1100: q_out[9:0] = 0b1010001110;
    1101: q_out[9:0] = 0b1001110001;
    1110: q_out[9:0] = 0b0101100011;
    1111: q_out[9:0] = 0b1011000011;
endcase;
```

#### Verilog Implementation

This code is derived from [fpga4fun's post](https://www.fpga4fun.com/HDMI.html) on HDMI.

We start by implementing the TMDS encoder for DVI, as mentioned above.

```verilog
module TMDS_encoder(
	input clk,
	input [7:0] VD,  // video data (red, green or blue)
	input [1:0] CD,  // control data
	input VDE,  // video data enable, to choose between CD (when VDE=0) and VD (when VDE=1)
	output reg [9:0] TMDS = 0
    );

    wire [3:0] Nb1s = VD[0] + VD[1] + VD[2] + VD[3] + VD[4] + VD[5] + VD[6] + VD[7];
    wire XNOR = (Nb1s>4'd4) || (Nb1s==4'd4 && VD[0]==1'b0);
    wire [8:0] q_m = {~XNOR, q_m[6:0] ^ VD[7:1] ^ {7{XNOR}}, VD[0]};

    reg [3:0] balance_acc = 0;
    wire [3:0] balance = q_m[0] + q_m[1] + q_m[2] + q_m[3] + q_m[4] + q_m[5] + q_m[6] + q_m[7] - 4'd4;
    wire balance_sign_eq = (balance[3] == balance_acc[3]);
    wire invert_q_m = (balance==0 || balance_acc==0) ? ~q_m[8] : balance_sign_eq;
    wire [3:0] balance_acc_inc = balance - ({q_m[8] ^ ~balance_sign_eq} & ~(balance==0 || balance_acc==0));
    wire [3:0] balance_acc_new = invert_q_m ? balance_acc-balance_acc_inc : balance_acc+balance_acc_inc;
    wire [9:0] TMDS_data = {invert_q_m, q_m[8], q_m[7:0] ^ {8{invert_q_m}}};
    wire [9:0] TMDS_code = CD[1] ? (CD[0] ? 10'b1010101011 : 10'b0101010100) : (CD[0] ? 10'b0010101011 : 10'b1101010100);

    always @(posedge clk) TMDS <= VDE ? TMDS_data : TMDS_code;
    always @(posedge clk) balance_acc <= VDE ? balance_acc_new : 4'h0;
endmodule
```

#### Conclusion

For a deep dive into implementing your own DVI/HDMI module, take a look at [this application note from Xilinx](/uploads/xapp460.pdf).

#### References

* [fpga4fun: TMDS encoder Verilog code](https://www.fpga4fun.com/HDMI.html)
* [Understanding HDMI](https://docs.google.com/document/d/1v7AJK4cVG3uDJo_rn0X9vxMvBwXKBSL1VaJgiXgFo5A)
* [ProjectF: HDMI Timing](https://projectf.io/posts/video-timings-vga-720p-1080p/)
* [Xilinx XAPP460: TMDS I/O in Spartan 3](https://www.xilinx.com/support/documentation/application_notes/xapp460.pdf)