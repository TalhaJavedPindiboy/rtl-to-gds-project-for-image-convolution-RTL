Project Overview: 3x3 Image Convolution RTL-to-GDSII
This repository illustrates the entire ASIC design flow including Register-Transfer Level (RTL) code to a manufacturable GDSII layout. The design is a 3x3 convolution engine, and is a stream convenience. targeted to a 45nm standard cell design library (gsclib045).


RTL Architecture (Verilog Design)
The frontend design adopts a spatial filtering module which is frequently used in digital image processing to perform spatial filtering, such as edge detection, sharpening or blurring.

Data Streaming & Line Buffers: Module receives a continuous flow of 8-bit image pixels. Rather, it makes use of internal shift registers that serve as line buffers to store incoming rows and therefore forms a real-time 3x3 pixel window.

Kernel & MAC Pipeline: A 3x3 programmable weight kernel to the active window. A multiply accumulate (MAC) pipeline multiply the 9 pixels in the window by their respective 9 kernel weights and adds the results.

Control Logic: A finite state machine (FSM) is used to implement a machine that tracks the "current" pixel coordinates and also controls the boundaries of the image, including padding/cropping, the delays in the pipeline, and asserts an out_valid signal when a resulting pixel is "ready.



2. Logic Synthesis (Cadence Genus)
Synthesis converts hardware-independent RTL Verilog logic to a technology-specific implementation of the gates that represent the logic in the netlist.

One of the tools is Library Mapping, which performs the mapping of logical operators (such as + or *) onto lib cells, such as ADDFHX1, NAND2X1 or DFF2RX1 that are available in the 45nm gsclib045 library.

Timing Constraints: A synopsys design constraints (sdc) file specifies the maximum frequency, maximum fan- out and maximum I/O delay.

Optimization: Genus structures the data-path, and minimizes the logic, resizing the gates up and down to guarantee all signals have met setup and hold timing constraints.

Handoff: Generates a structural Verilog netlist (image_conv_top_netlist.v) and updates an existing .sdc file for the physical design team.





3.Physical Design (Cadence Innovus)
The backend flow is applied to the synthesized netlist and converts it to a physical silicon layout, placing cells and routing the metal tracks and following semiconductor manufacturing rules.

Floorplanning & Pin Assignment: Core area & standard cell rows specified. This is a block-level design with no physical bond-pad macros, thus the 92 signal I/O terminals are split up around all four edges in the form of perimeter metal pins.

Power Planning (PP): Make up a power delivery network to avoid drop in voltage (IR drop). Attached to the core, the tracks Metal10 and Metal11 create two power rings, called VDD and VSS, adjacent to the core, and with vertical and horizontal stripes which drop down to power the normal rows of cells working to FollowPins.

Placement: Allows all 2,214 standard cells to be placed in the rows of a floorplan. It clusters cells with high connectivity closely to minimize overall wire-length and routing congestion.

Clock Tree Synthesis (CTS): A dedicated routing step creates a good clock distribution tree. To minimise any clock skew, Innovus includes clocks buffers such as CLKBUFX12 for the clock to arrive to all 152 Flip Flops at the same time.

Routing: NanoRoute routes all standard cells with 0-overlap with obstacles, respecting strictly the minimum spacing and width (45nm Design Rule Check – DRC).

After inserting the filler cells to maintain uniform silicon density the design goes through DRC and Layout Versus Schematic (LVS) to determine if there are any discrepancies. The final physical layout is saved out as an image_conv_top.gds file.
