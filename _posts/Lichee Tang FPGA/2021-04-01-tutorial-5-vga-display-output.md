---
layout: post
title: 'Tutorial 5: VGA Display Output'
menus: ''
date: 2021-04-01 16:00:00 +0000

---
### VGA Display Output

VGA is an older display interface based on analog inputs of Red, Green and Blue for each pixel. The data for the pixels are changed by toggling `HSYNC` and `VSYNC` to indicate the active display area. The clock rate is defined by the resolution of the display output. A handy reference for VGA is available at [Digikey](https://www.digikey.sg/eewiki/pages/viewpage.action?pageId=15925278). Most of this tutorial is based on information from that blog post.

![](/uploads/vga_signal_timing_diagram.jpg)

| Resolution | Refresh Rate | Pixel Clock (MHz) | Display (H) | Inactive Area (H) | Display (V) | Inactive Area (V) |
|---         |---           |---                |---          |---                |---          |---                |
| 640x480    | 60           | 25.175            | 640         | 160               | 480         | 120               |

To 