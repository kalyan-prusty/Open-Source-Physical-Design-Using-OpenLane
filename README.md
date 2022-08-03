# Open-Source-Physical-Design-Using-OpenLane
This is an interactive tutorial that i have done during the VSD Advanced Physical Design workshop using OpenLANE.

OpenLane is a opensource RTL to GDSII flow which has several components starting from OpenROAD, Yosys etc to  Fault, Magic, Netgen and many more that will be discussed in the future sections of this repository. The RTL to GSDII flow consists of following steps shown in the following image.
![image](https://user-images.githubusercontent.com/33130256/182671443-4abc6400-5661-44dd-ab7b-9a6ecf0c28f2.png)

## Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

## Day 2 - Good floorplan vs bad floorplan and introduction to library cells
### Theory
##### Core Vs Die
A 'core' is the section of the chip where the fundamental logic of the design is placed.Where as a die, which consists of core, is small semiconductor material specimen on which the fundamental circuit is fabricated. The following image will make your idea clear.
![image](https://user-images.githubusercontent.com/33130256/182674867-834df35d-7188-495e-a720-48bca9cb3b09.png)

##### Utilization Factor
Utilization Factor is ratio of the area occupied by netlist [without routing or clock tree only the standard cell area] to the total core area.The utilization factor is generally kept in the range of 50% - 60% to facilitates routing, clock tree etc.

##### Aspect Ratio
Aspect ratio specifies the shape of your chip [not only standard cell], and is defined by the height of the core area divided by the width of the core area.

##### Preplaced Cells
Some of designes are only once desiged and repeatedly used in the design. Some of these IP's are Memory Cells, Clock-gating Cells etc. These blocks have user-defined locations, and hence are placed in chip before automated placemnet and routing tools. Hence are called preplaced cells. 

##### Decoupling Capacitor
Decoupling capacitors are placed near to preplaced cells for proper transfer/communication of logic between them as shown in following image. This will charge up to the power supply voltage over time and it will work as a charge reservoir when a transition is needed by the circuit instead of the charge coming from the power supply which may face issue of RL drop over the interconnect wires.
![image](https://user-images.githubusercontent.com/33130256/182683886-42dde8ea-3f6d-4591-8f22-0e185b610e53.png)



