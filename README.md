# Open-Source-Physical-Design-Using-OpenLane
This is an interactive tutorial that i have done during the VSD Advanced Physical Design workshop using OpenLANE.

## Overview of Physical Design flow
Place and Route (PnR) is the core of any ASIC implementation and Openlane flow integrates into it several key open source tools which perform each of the respective stages of PnR.
Below are the stages and the respective tools (in ( )) that are called by openlane for the functionalities as described:
- Synthesis
  - Generating gate-level netlist ([yosys](https://github.com/YosysHQ/yosys)).
  - Performing cell mapping ([abc](https://github.com/YosysHQ/yosys)).
  - Performing pre-layout STA ([OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)).
  ![image](https://user-images.githubusercontent.com/33130256/183292584-15ee6cb3-a00e-4bca-969d-f20251af47b0.png)
  
- Floorplanning
  - Defining the core area for the macro as well as the cell sites and the tracks ([init_fp](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/init_fp)).
  - Placing the macro input and output ports ([ioplacer](https://github.com/The-OpenROAD-Project/ioPlacer/)).
  - Generating the power distribution network ([pdn](https://github.com/The-OpenROAD-Project/pdn/)).
  
![image](https://user-images.githubusercontent.com/33130256/183292613-a8c28bfa-9bd2-4e07-a9a6-96d777c1b7cc.png)
![image](https://user-images.githubusercontent.com/33130256/183292684-7a5c5814-e37a-4b32-8fe1-37831acf3ab0.png)

- Placement
  - Performing global placement ([RePLace](https://github.com/The-OpenROAD-Project/RePlAce)).
  - Perfroming detailed placement to legalize the globally placed components ([OpenDP](https://github.com/The-OpenROAD-Project/OpenDP)).
 ![image](https://user-images.githubusercontent.com/33130256/183292648-de25e194-583f-4949-b2af-2e268ad9266e.png)
 
- Clock Tree Synthesis (CTS)
  - Synthesizing the clock tree ([TritonCTS](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/TritonCTS)).
 ![image](https://user-images.githubusercontent.com/33130256/183292782-8621d0ff-15b6-4808-a5d6-e58430cbb4d4.png)

- Routing
  - Performing global routing to generate a guide file for the detailed router ([FastRoute](https://github.com/The-OpenROAD-Project/FastRoute/tree/openroad)).
  - Performing detailed routing ([TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute))
![image](https://user-images.githubusercontent.com/33130256/183292795-5ac2b722-75f5-49e9-8676-bf6814020bb3.png)

- GDSII Generation
  - Streaming out the final GDSII layout file from the routed def ([Magic](https://github.com/RTimothyEdwards/magic)).

## Overview of OpenLane
OpenLane is a opensource RTL to GDSII flow which has several components starting from OpenROAD, Yosys etc to  Fault, Magic, Netgen and many more that will be discussed in the future sections of this repository. The RTL to GSDII flow consists of following steps shown in the following image.
![image](https://user-images.githubusercontent.com/33130256/182671443-4abc6400-5661-44dd-ab7b-9a6ecf0c28f2.png)

## Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
Invoke openlane by the following command in the openlane folder. Before that you have enabled docker.
```
docker
```
A custom shell script or commands can be generated to make the task simpler.

To invoke OpenLANE run the ```./flow.tcl``` script. To use interactive mode use -interactive flag with ./flow.tcl.

``` 
./flow.tcl -interactive 
```
### Lab -1
The first step after invoking OpenLANE is to import the openlane package of required version. This is done using following command. Here 0.9 is the required version of OpenLANE.
```
package require openlane 0.9
```

##### Design Preparation

The next step is to prepare our design for the OpenLANE flow. This is done using following command:
```
prep -design <design-name>
````
Some additional flags that can be used while preparation are:

```-tag <name-for-current-run>``` - All the files generated during the flow will be stored in a directory named <name-for-current-run>
  
```-overwrite``` - If a directory name mentioned in -tag already exists, it will be overwritten.

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/183296023-2ea2019e-2b54-4894-97a2-923094f7d60f.png" width=50% height=50%>
</p>
  
While doing this we merged the technology lef [contains info about metal layer vias etc] and standard cell lef  information together to make a merged.lef file which is located in <design_folder>/runs/tmp/ .

config.tcl in the run tells us about the parameter that needs to be taken by the run. Basically sets a bunch of environment variables.

next step to run_synthesis with yosys and abc scripts. For synthesizing the design run the following command.
```
run_synthesis  
```
After some time you will see the success window infront of you.
<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/183300612-00d9d1d7-e351-4698-aba2-12bde3766d00.png" width=50% height=50%>
</p>

Now let's count the total number of cells and flops to find the flop ratio.

  <table border="0">
   <tr>
    <td> <img src="https://user-images.githubusercontent.com/33130256/183300828-80dea6df-2064-42ca-b7e3-f798e84f0b5a.png"> </td>
    <td> <img src="https://user-images.githubusercontent.com/33130256/183300837-d997a772-96a7-4916-868f-836b9853f7a5.png"> </td>
   </tr>
  </table>
Total number of Cells: 14876
Total number of DFF: 1613

Flop Ratio: 1613 / 14876 = 0.108

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

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182683886-42dde8ea-3f6d-4591-8f22-0e185b610e53.png" width=50% height=50%>
</p>

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

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182763186-06558fb9-f749-42e0-82e6-54d7a3d31719.png" width=50% height=50%>
</p>

We can check the DEF(Design Exchage Format) file for more information related to floorplan. A DEF file contains the design-specific information of the circuit and it is a representation of the design at any point during the physical design. This looks like this 

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182775077-8817b10d-e166-4c06-a415-123ffe0ad0b1.png" width=50% height=50%>
</p>


Here we can see that the yellow highlighted area shows the actual area of my die which is 660.685 µm X 671.405 µm. 

##### Viewing Floorplan in Magic
To view our floorplan in Magic we need to provide three files as input:

- Magic technology file (sky130A.tech)
- Merged LEF file (cell lef + tech lef)
- Def file of floorplan that we saw after running floorplan

```
magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```


<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182835028-46fc9248-3190-498c-bda4-facebf2de879.png">
</p>

If evrything works fine then the following window will open.

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182837264-897e8cbf-1ae5-461d-b67e-a11fcd3fbc3d.png" width=50% height=50%>
</p>


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

Again run the same magic command with the def file location changed to def file that is being generated after placement.

```
magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```
If everything goes fine then the following layout is shown.

![image](https://user-images.githubusercontent.com/33130256/182867679-c2e31181-91ac-4382-a1f6-670ccc83d1ad.png)

### Theory - 3
##### Standard Cell Library 
This contains all the information like area, delay, threshold voltage & power consumption etc about different gates, flipflops etc along with their different sizes i.e. different drive strength. 

##### Cell Design Flow

<p align="center" width="100%">
    <img src="" width=50% height=50%>
</p>

<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182932162-bea087d5-1078-48d9-821c-0cc420d78d81.png">
</p>

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
Clone the repo from https://github.com/nickson-jose/vsdstdcelldesign.git. Run the meg file in magic software using the following command
```
magic -T sky130A.tech sky130_inv.mag
```
<p align="center" width="100%">
    <img src="https://user-images.githubusercontent.com/33130256/182999348-caf44fd9-30d4-443f-bdcf-435666250456.png" width=50% height=50%>
</p>

### Theory - 2
##### 16-Mask CMOS Process
- Selecting a substrate

![image](https://user-images.githubusercontent.com/33130256/183001839-f623db13-f5d0-47b2-b15b-a88c972aa37f.png)
- Create an active region for transistors

![image](https://user-images.githubusercontent.com/33130256/183001387-bd6bdb3d-1881-4aa3-9be7-f669a83aaad5.png)
- Nwell & Pwell formation using Twin Tub Process

![image](https://user-images.githubusercontent.com/33130256/183002903-84dc7385-cb63-4fc0-8cb3-0ea6eda6fbfd.png)
- Formation of Gate terminal using polysilicon layer over the oxide & 

![image](https://user-images.githubusercontent.com/33130256/183004080-6787acb9-d392-49ac-8e36-712dd5bc3bfc.png)
- LDD (Lightly Doped Drain) Formation to avoid hot electron effect and short channel effect 

![image](https://user-images.githubusercontent.com/33130256/183010085-9dac6a95-071b-4273-bd2f-30345046f6d2.png)
- Source and Drain Formation 

![image](https://user-images.githubusercontent.com/33130256/183011905-c1229a5d-c943-41e5-97a2-b1274ae4aa9d.png)
- Contact and Interconnect Formation

![image](https://user-images.githubusercontent.com/33130256/183013021-7b694376-d230-4d63-871b-df6cfbdb8a44.png)
- Higher Level metal layer Formation:

![image](https://user-images.githubusercontent.com/33130256/183015775-04630edc-c22a-4f1b-9f3a-714a8d85d2e5.png)

### Lab - 3
##### Extract SPICE Netlist from Layout in Magic
This is done in 2 steps.

Create the extraction file using 
```
extract all
```
This will create a .ext file and using which we can generate the spice file using the following commands to be used in ngspice for simulation.
```
ext2spice cthresh 0 rthresh 0
ext2spice
```
![image](https://user-images.githubusercontent.com/33130256/183026065-1307a83c-2c1d-4268-966a-74ab932c30fb.png)

and from the above image it is confirmed that we have the .spice file with us.
![image](https://user-images.githubusercontent.com/33130256/183027987-d44be707-eef7-448f-87ec-ddf24a8fee84.png)

After this edit the spice file to include the relevent library, sources and command to run thew simulation/
```
.option scale= 0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

//.subckt sky130_inv A Y VPWR VGND
M0 Y A VGND VGND nshort_model.0 ad=0 pd=0 as=0 ps=0 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0  ad=0 pd=0 as=0 ps=0 w=37 l=23

VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

C0 A VPWR 0.07fF
C1 A Y 0.05fF
C2 Y VPWR 0.11fF
C3 Y VGND 0.24fF
C4 VPWR VGND 0.59fF
//.ends


.tran 1n 20n

.control
run
.endc
.end
```

If everything goes fine then the following window will pop up asking for ngspice command.

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183133768-06e13727-0820-41e8-97e8-e6a26ac73bfb.png"> 
</p>

Execute plot command to observe the following plot
![image](https://user-images.githubusercontent.com/33130256/183133312-e8dfc8f6-470b-4d4e-9796-84c8215c6e28.png)

Now the timing charecterization can be easily calculated.
1. Fall Delay: Time between input rise to output fall

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183134230-fe059e6c-6d65-4e65-9a11-307e98844f01.png"> 
</p>

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183134460-09b2ed60-0374-4f29-b2cd-45f55a9122ac.png"> 
</p>

```
tdf = 0.003 ns
```

2. Rise Delay: Time between input fall to output rise

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183135025-e3cb6504-3b94-47cd-866f-e346da967975.png"> 
</p>

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183135112-a652c18d-cb69-4c6a-b902-f4b8b1c28656.png"> 
</p>

```
tdr = 0.028 ns
```

3. Fall Time: Time required for the output to go from 20% to 80% of VDD

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183138421-8d04354c-80e9-42e8-aa8e-1d1fe89c1f98.png"> 
</p>

```
tr = 0.027 ns
```
4. Rise Time: Time required for the output to go from 80% to 20% of VDD

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183138011-c67393e5-b9de-4785-a330-4dbf7eb90759.png"> 
</p>

```
tr = 0.045 ns
```

This is how the timing charecterization is done for a standard cell.

## Day 4 - Pre-layout timing analysis and importance of good clock tree
##### Lab - 1
##### Magic to Standard Cell LEF Generation
PnR is possible just by giving information about the pin placement and metal information, there is no need of providing any information about the logic. This is done by the LEF file (Library Exchange Format) to perform interconnect routing in conjunction to routing guides generated from the PnR flow. This is how the companies do not disclose the logic information to the foundry. 

Before generating the LEF file for our standard cell design we need to ensure that the design we have made is satisfing the foundry requirment i.e. track details. This we can confirm by making a grid in magic with the proper details of the tracks from track.info file as shown below.
![image](https://user-images.githubusercontent.com/33130256/183233728-7fbe1a5b-8c89-4a4d-9897-4e0cc9bc183f.png)
The format followed above is 
```
<layer-name> <X/Y direction> <track-offset> <track-pitch>
```

The same info has to be passed to magic tool to form the grid.To create a standard cell LEF from an existing layout, some important aspects need to be taken into consideration.

- The input and ouptut of the cell fall on intersection of the vertical and horizontal tracks (grid lines).

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183236748-95317a3a-d4c8-42a1-87e5-01eda2cfb4b9.png"> 
</p>


- The height of cell  should be an odd multiple of the vertical track pitch [mentioned in the track.info file].
- The width of cell should be an odd multiple of the horizontal track pitch [mentioned in the track.info file].

<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183237687-8e7c7fb6-256c-4c13-9eef-8d79d9367493.png"> 
</p>

In the above image we can see the inner rectangle which represents the boundary for PnR and we can confirm from the image that the width and hight of the red rectangle  are odd multiple of x & y pitch respectively.

After the layout of the user-defined cell is made we have to defining port and set correct class and use attributes to each port is the first step which was already done [here](https://github.com/nickson-jose/vsdstdcelldesign/blob/master/README.md). Once the properties are defined then we can go for generation of lef file as follows.

```
lef write sky130_kalyaninv.lef
```
After running the command you will be left with a LEF file which is shown below.
<p align="center" width="100%">
    <img width="50%" src="https://user-images.githubusercontent.com/33130256/183238421-8c41a23e-3123-4228-8124-bc1eff82ddfd.png"> 
</p>

now change the config.tcl file for your project to include the lef file you have made and along with that set the library to include the new standard cell information.After changing the config.tcl file it looks somewhat like

```
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) "./designs/picorv32a/src/picorv32a.v"
set ::env(SDC_FILE) "./designs/picorv32a/src/picorv32a.sdc"

set ::env(CLOCK_PERIOD) "5.000"
set ::env(CLOCK_PORT) "clk"


set ::env(CLOCK_NET) $::env(CLOCK_PORT)

set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]


set filename $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1} {
        source $filename
}
```
Now, let's re run the synthesis and check. So folowing the same steps as explained in Day-2 labs we can run the synthesis for same picorvb32a project but with our inv.

<table border="0">
   <tr>
    <td> <img src="https://user-images.githubusercontent.com/33130256/183240573-27cf40fc-b043-41a2-bc75-5b17838943bd.png"> </td>
    <td> <img src="https://user-images.githubusercontent.com/33130256/183256892-d899f2b9-7616-4631-b117-16e706b22c5d.png"> </td>
   </tr>
   </table>

Now, we can see this is violating the timing. 

##### Timing Analysis using OpenSTA
Steps to remove this timing violation:
1. Synthesize your design keeping Delay as optimizing criteria insteade of Area i.e. changing the SYNTH_STRATEGY
2. Enable sizing.
3. Decrease the maximum fanout so that output capacitance will be decreased and inturn the delay.

All the above changes are made in the synthesis.tcl file located in the openlane/configuration folder.

![image](https://user-images.githubusercontent.com/33130256/183260020-50d467df-746d-43e6-861f-47c6f135aa5d.png)

Now run synthesis again and observe that both wns and tns is redeuced to 0. This same can be confirmed using OpenSTA again.
make a pre_sta.conf file and put content equivalent to following content
```
set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
  
read_liberty -min /home/kalyanprusty/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib
read_liberty -max /home/kalyanprusty/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_verilog /home/kalyanprusty/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-08_10-12/results/synthesis/picorv32a.synthesis.v
link_design picorv32a
read_sdc /home/kalyanprusty/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc

report_checks -path_delay min_max -fields {slew trans net cap input_pin}
report_tns
report_wns
         
```
Now run the following command
```
sta pre_sta.conf
```
you can observe following results
![image](https://user-images.githubusercontent.com/33130256/183260551-c122a5e0-d13c-4cca-8099-d07661fd461d.png)

After this we can run floorplan and placement as done before to see the placed design in the magic again.

##### Clock Tree Synthesis
Run the following command to run the CTS in the previous design.
```
run_cts
```
In the synthesis folder we can see another new netlist as follows:
![image](https://user-images.githubusercontent.com/33130256/183262420-7d033080-4d25-4462-88a5-034d8f2e24d0.png)
Both the netlist files are different as the one before CTS does not contain the clock b uffers but the one aftyer cts has all the clock buffer information as well.

##### Post-CTS STA Analysis
OpenLANE has the OpenROAD application integrated into its flow. The OpenROAD application has OpenSTA integrated into its flow. Therefore, we can perform STA analysis from within OpenLANE by invoking OpenROAD. So in the openlane execute the following command to open openroad in openlane
```
openroad
```

Read lef and def files after CTS and make the db.
1. Reading LEF:
```
read_lef /openLANE_flow/designs/picorv32a/runs/03-08_10-12/tmp/merged.lef
```
2. Reading DEF:
```
read_def /openLANE_flow/designs/picorv32a/runs/03-08_10-12/results/cts/picorv32a.cts.def
```
lef file will not change untill you change the technology but the def files change depending on how you add different cells to your design.
3. Writing a DB:
```
write_db pico_cts.db
```
4. Read the same DB:
```
read_db pico_cts.db
```
5. Read the post_cts netlist:
```
read_verilog /openLANE_flow/designs/picorv32a/runs/03-08_10-12/results/synthesis/picorv32a.synthesis_cts.v
```
6.1. Set the FAST and SLOW library
```
read_liberty -max $::env(LIB_SLOWEST)
read_liberty -min $::env(LIB_FASTEST)
```
(or)
6.2. Set the Typical library
```
read_liberty  $::env(LIB_SYNTH_COMPLETE)
```
7. Read sdc
```
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
```
8. Set propagated clocks
```
set_propagated_clock [all_clocks]
```
9. Check the reports
```
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```
The above process checks the design taking the real clocks into consideration. After this if the thing go correct then timings met will be shown.

![image](https://user-images.githubusercontent.com/33130256/183276403-5177d5fc-20f0-4af3-9cc0-ae3a3a186fca.png)

## Day 5 - Final steps for RTL2GDS using tritonRoute and openSTA

##### Global and Detailed Routing 
OpenLANE uses FastRoute as global and TritonRoute as the detailed routing engine for physical implementations of designs. Routing consists of two stages:

Global Routing - first partitions the routing region into tiles and decides tile-to-tile paths for all nets.
Detailed Routing - the exact tracks and vias for nets after the routing guide is provided.

In openlane these all steps are done by simple execution of following command
```
run_routing
```
The follwoing image shows routing has been completed without any DRC violation.

![image](https://user-images.githubusercontent.com/33130256/183288422-3721236c-0b1a-4923-9497-1f6079f65b7d.png)

![image](https://user-images.githubusercontent.com/33130256/183288477-bd7397e6-ea5e-4bc8-9ff4-341dd08aed47.png)

##### SPEF Extraction
After routing we need to get the information about the parasitic information of the routed path as these will also impact the STA analysis. SPEF file is needed to perform sign-off post-route STA analysis. SPEF Extractor has been included in the openlane flow. Following image shows that SPEF file has been generated. There is no need to execute any specific command for this extraction. This file is automatically being generated at the end of routing process.

<table border="0">
   <tr>
    <td> <img src="https://user-images.githubusercontent.com/33130256/183288991-fbd484d1-e44e-43c0-97a9-05c24588b4f3.png"> </td>
    <td> <img src="https://user-images.githubusercontent.com/33130256/183289146-4e31861c-d177-439f-8d46-2cc9e810e4aa.png"> </td>
   </tr>
</table>
   
## References
  - [VLSI System Design](https://www.vlsisystemdesign.com/)
  - [OpenLANE](https://github.com/The-OpenROAD-Project/OpenLane)
 
## Acknowledgements
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder (VSD Corp. Pvt. Ltd)
- [Nickson Jose](https://github.com/nickson-jose)
- [ShonTaware](https://github.com/ShonTaware)
- [Grant Brown](https://gitlab.com/gab13c)
