# Open-Source-Physical-Design-Using-OpenLane
This is an interactive tutorial that i have done during the VSD Advanced Physical Design workshop using OpenLANE.

OpenLane is a opensource RTL to GDSII flow which has several components starting from OpenROAD, Yosys etc to  Fault, Magic, Netgen and many more that will be discussed in the future sections of this repository. The RTL to GSDII flow consists of following steps shown in the following image.
![image](https://user-images.githubusercontent.com/33130256/182671443-4abc6400-5661-44dd-ab7b-9a6ecf0c28f2.png)

## Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

## Day 2 - Good floorplan vs bad floorplan and introduction to library cells
### Theory - 1
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

##### Ground Bounce and Voltage Droop
When there is a sudden change in current supply at that particular point the voltage of that point increase or decrease rapidly depending on the direction of current. As change in current is directly proportional to voltage in inductor [electrical equivalent of wire] this can result in a voltage change at particular time. This is called **ground bounce**. The same is also seen at the source/ supply which is called **supply droop**. During charging of all the capacitor at a time multiple capacitors can not be charged to Vdd in 0 time which is seen as a drop in voltage level. This is called **supply/voltage droop**. Things become unpredictable if these levels cross the nise margine level.

##### Power Planning
The problem of Ground and supply bounce can be removed if if we have multiple power supply or sink for charging and discharging respectively. Power planning is a step in which power grid network is created to distribute power and ground to each part of the design symmetrically. This will reduce IR drop and charge accumulation at a particular ground resulting high noise margine and signal integity. 

##### Pin Placement
Pin placement decides timing delays and number of buffers required in the whole core which inturn decreases the power consumption as number of buffer used decreased. In general the input pins which are inputs of a particular block are placed near the block. Clock nets are thicker than the normal routing wire as these wires mainly drive the whole design continuously hence a low resistance path is required. Before placement and routing we logically block the space in the I/O ring area which discriminate core and I/O area.

So in floor planning step we are going to set the core & die area, set aspect ratio, plan the power grid, place the pins of our design.

### Lab - 1
After the synthesis we can do the floor planning by running the following command in openlane.
```
%run_floorplan
```
On successfull run of the command the terminal looks as:
![image](https://user-images.githubusercontent.com/33130256/182763186-06558fb9-f749-42e0-82e6-54d7a3d31719.png)

We can check the DEF(Design Exchage Format) file for more information related to floorplan. A DEF file contains the design-specific information of the circuit and it is a representation of the design at any point during the physical design. This looks like this 
![image](https://user-images.githubusercontent.com/33130256/182775077-8817b10d-e166-4c06-a415-123ffe0ad0b1.png)

Here we can see that the yellow highlighted area shows the actual area of my die which is 660.685 µm X 671.405 µm. 

##### Viewing Floorplan in Magic
To view our floorplan in Magic we need to provide three files as input:

- Magic technology file (sky130A.tech)
- Merged LEF file (cell lef + tech lef)
- Def file of floorplan that we saw after running floorplan

```
magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```
![image](https://user-images.githubusercontent.com/33130256/182835028-46fc9248-3190-498c-bda4-facebf2de879.png)

If evrything works fine then the following window will open.

![image](https://user-images.githubusercontent.com/33130256/182837264-897e8cbf-1ae5-461d-b67e-a11fcd3fbc3d.png)

This following image shows that the horizontal pins are in 3rd metal layer.

![image](https://user-images.githubusercontent.com/33130256/182839954-508fcab3-1e77-4485-8f31-d6b88e395d2f.png)

### Theory - 2
##### Placement
After this we bind the netlist of our design with the standard cells provided by the foundary in a library format. This library contains all the information about the cell like width, hight and delay information etc. Standard cells will be placed on the floorplan according to the given netlist without disturbing the preplaced cells in floorplan. The delay that will be incured is taken into consideration while placing the cells.

##### Repeaters
This is cell which takes the original signal, reproduce it and send it to the output. These are used for maintaining the signal integrity between the cells placed at a sufficient distance from eachother. These are added in the optimizing the placement where we estimate wire length and capacitance and, based on that repeaters are inserted. This divides the long distance into multiple shorter distances.

After this stage setup timing analysis is done with the ideal clock (there is no delay for the clock to reach from one point in the core to another point) and standard cells & repeaters placed on the floorplan.

### Lab - 2
OpenLANE does placement in two stages:

1. Global Placement - Optimized but not legal placement [Corase Placement]. Optimization works to reduce wirelength by reducing half parameter wirelength [HPWL]
2. Detailed Placement - Legalizes placement [Fine Placement] of cells into standard cell rows while adhering to global placement

Legalization in placement means the standard cells should be place in a row and should not overlap eachother and should be close to eachother. In openlane run the following command:
```
%run_placement
```
![image](https://user-images.githubusercontent.com/33130256/182866007-dd3a5a1d-22f9-459b-90cf-bcb848c6c26b.png)

Again run the same magic command with the def file location changed to def file that is being generated after placement.

```
magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```
If everything goes fine then the following layout is shown.

![image](https://user-images.githubusercontent.com/33130256/182867679-c2e31181-91ac-4382-a1f6-670ccc83d1ad.png)

Note: Generally Power planning is done in the floorplan stage but in openlane the sequence is little bit different and the power planning is done after CTS stage.

### Theory - 3
##### Standard Cell Library 
This contains all the information like area, delay, threshold voltage & power consumption etc about different gates, flipflops etc along with their different sizes i.e. different drive strength. 

##### Cell Design Flow
![image](https://user-images.githubusercontent.com/33130256/182932162-bea087d5-1078-48d9-821c-0cc420d78d81.png)

The above image shows the standard cell design flow and what are the steps involved in finding out the cell related information. All the input, design steps and output of the steps are mentioned above. All the input data is used in most of the design steps.

CDL (Circuit Description Language) file is the outpur of Circuit design step. Similarlly layout design provides us GDSII [to be used by the foundry], LEF[whole layout information], CIR file (Extracted spice Netlist). Timing, noise and power related information about the cell is derived in the charecterization step in the .lib format.

## Day 3 - Design library cell using Magic Layout and ngspice characterization

### Lab - 1
All the variables/switches can be changed on the way while running openlane. OpenLane uses one tool named IOPlacer for proper placement of the IO pins arround the chip. After changing the IOplacer format the floorplan will look like
![waveform](https://user-images.githubusercontent.com/33130256/182995556-b4ce6cc4-28ae-433e-bd86-22e4c861c0f8.png)

RED: I/O pins are present
GREEN: I/O pins are not present

### Theory - 1
##### CMOS Inverter 
CMOS cells have five modes of operation:

- NMOS Cutoff PMOS Linear 
- NMOS Saturation PMOS Linear 
- NMOS Saturation PMOS Saturation 
- NMOS Linear PMOS Saturation 
- NMOS Linear PMOS Cutoff 

Thershold voltage is the voltage at which Vin = Vout. Threshold voltage is a function of the W/L ratio of a device, therefore varying the W/L ratio will vary the output waveform of CMOS devices and inturn the transfer charecteristic of the device as well. A perfectly symmetrical device will have a switching threshold such that Vin = Vout = VDD/2 which is achieved when (W/L)p is approximatly 2.5 times (W/L)n.


### Lab - 2

##### Magic Layout View of Inverter Standard Cell


