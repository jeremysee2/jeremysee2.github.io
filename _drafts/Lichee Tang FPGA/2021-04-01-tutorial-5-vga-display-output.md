---
layout: post
title: 'Tutorial 5: VGA Display Output'
date: 2021-04-01T16:00:00.000+00:00
menu:
  '': {}

---
### VGA Display Output

VGA is an older display interface based on analog inputs of Red, Green and Blue for each pixel. The data for the pixels are changed by toggling `HSYNC` and `VSYNC` to indicate the active display area. The clock rate is defined by the resolution of the display output. A handy reference for VGA is available at [Digikey](https://www.digikey.sg/eewiki/pages/viewpage.action?pageId=15925278). This tutorial is based on information from that blog post, as well as the [relevant Nandland tutorial](https://www.youtube.com/watch?v=7wjTJivsNMM).

![](/uploads/vga_sync.png)

| Resolution | Refresh Rate | Pixel Clock (MHz) | Display (H) | Inactive Area (H) | Display (V) | Inactive Area (V) |
| --- | --- | --- | --- | --- | --- | --- |
| 640x480 | 60 | 25.175 | 640 | 160 | 480 | 120 |

From this, we know that we need to manage the VSYNC and HSYNC signals to control when we output RGB pixel data. Let's start by creating a module that generates the appropriate VSYNC and HSYNC signals, given a certain fixed resolution. Create the following file `VGA_Sync_Pulses`.

```v
// This module is designed for 640x480 with a 25 MHz input clock.

module VGA_Sync_Pulses 
 #(parameter TOTAL_COLS  = 800, 
   parameter TOTAL_ROWS  = 525,
   parameter ACTIVE_COLS = 640, 
   parameter ACTIVE_ROWS = 480)
  (input             i_Clk, 
   output            o_HSync,
   output            o_VSync,
   output reg [11:0] o_Col_Count = 0, 
   output reg [11:0] o_Row_Count = 0
  );  
  
  always @(posedge i_Clk)
  begin
    if (o_Col_Count == TOTAL_COLS-1)
    begin
      o_Col_Count <= 0;
      if (o_Row_Count == TOTAL_ROWS-1)
        o_Row_Count <= 0;
      else
        o_Row_Count <= o_Row_Count + 1;
    end
    else
      o_Col_Count <= o_Col_Count + 1;
      
  end
	
  // Only high in the ACTIVE AREA of the display
  assign o_HSync = o_Col_Count < ACTIVE_COLS ? 1'b1 : 1'b0;
  assign o_VSync = o_Row_Count < ACTIVE_ROWS ? 1'b1 : 1'b0;
  
endmodule
```

t