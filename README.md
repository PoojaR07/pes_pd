# Advanced Physical Design using OpenLANE/Sky130 

# COURSE
<details>
<summary>DAY 1 : Inception of opensource-EDA, Opennlane and Skywater130</summary>
  
## How to Talk to Computers
- First we look at the introduction to the RISC-V ISA(Instructiion Set Architecture). Supposing we need to execute a C program on a particular hardware. First the C-program is converted into Assembly Code( here for RISC-V processor). Then the assembly code is converted into binary. An RTL implements this code for the particular layout of the RISC-V processor and the output is visible.
- An application running on a system is usually written with the help of a high level language such as C,C++,Python etc. The code of these applications are compiled with the help of compilers running on a system software(OS). The compiler converts the high level code into assembly intructions for the particular processor. The assembler then converts the instructions into binary which is fed into the layout of the chip that processes every pattern of bits and the program is hence run.

## SoC Design and OpenLANE

**What is a PDK?**
- PDK stands for Process Design Kit.
- It is a collection of files used to model a fabrication process for the EDA tools used to design an IC
  - Process Design Rules.
  - Device Models
  - Digital Standard Cell Libraries
  - I/O Libraries
 
A simplified RTL to GDSII Flow is :
- Synthesis -> Floor/Power Planning -> Placement -> Clock Tree Synthesis -> Routing -> Signoff

- Synthesis - Converts RTL to a ciruit, out of compomments from the standard cell library.
- Floor and Power Planning - Obejctive here is to plan the silicon area and create robust power distribution network to power the chip.
  - Chip Floor Planning - Partition the chip die between different system building blocks and place the I/O pads.
  - Macro Floor Planning - We define the macro dimensions, pin locations and rows are defined.
  - Power Planning - The power distribution network is contructed.
- Placement - Placing the cells on the floorplan rows, aligned with the sites. There are 2 steps: Global and Detailed.
- Clock Tree Synthesis - To deliver the clock to all sequential elements.
- Routing - Implement the interconnect using the available metal layers.
- Sign Off - Perform physical verification such as DRC(Design Rule Check) and LVS(Layout vs Synthesis). Also perform STA(Static Timiing Analysis).

**OpenLANE ASIC Flow**

![Screenshot from 2023-09-10 23-56-09](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/b1bbef29-0748-4fd7-acf8-8c421d599aca)

## Getting Familiar with the Open Source EDA Tools
**Design Preparation Step**

![Screenshot from 2023-09-10 21-19-08](https://github.com/PoojaR07/pes_pd/assets/135737910/d0c066de-2b4c-4eb8-a7f0-ea21afea24d5)
```
cd openlane_working_dir/openlane/
```
- We now type the command ```docker```.
- This will open the shell as shown in the figure above
- Now we type
```
./flow.tcl -interactive
```
- Now we must import all the packages required to run the flow, we use the command:
```
package require openlane 0.9
```
- Now we do the design setup stage using the command:
```
prep -design picorv32a
```
![Screenshot from 2023-09-10 22-18-19](https://github.com/PoojaR07/pes_pd/assets/135737910/65a9cfa3-18af-4253-8f55-8ddc2eed2602)

![Screenshot from 2023-09-10 21-49-36](https://github.com/PoojaR07/pes_pd/assets/135737910/32897f23-78f3-4b5c-b905-d4cd3382cffd)
- To synthesize the design we type
```
run_synthesis
```
![Screenshot from 2023-09-17 11-55-33](https://github.com/PoojaR07/pes_pd/assets/135737910/4631ecd7-54bb-4483-b2c7-6aab270579ad)

- A synthesis successful message must be displayed.
![Screenshot from 2023-09-17 12-21-31](https://github.com/PoojaR07/pes_pd/assets/135737910/f00ffe53-6d6e-40b9-9d71-35fe4d372edd)

- The flop ratio can be calculated by using:
```
No. of flops/No. of cells = 1613/14876 = 0.108
```
- In percentage there is 10.8% of the total number of cells are Flops
![Screenshot from 2023-09-17 12-18-38](https://github.com/PoojaR07/pes_pd/assets/135737910/13785578-1a53-413f-a7a0-068f521c0b8c)


</details>
<details>
<summary>DAY 2 : Good Floorplan vs Bad Floorplan and Introduction to library cells</summary>

**Utilization Factor and Aspect Ratio**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/a1812369-af71-48c4-860c-f76af506400e)
- We consider a simple netlist with a Launch and Capture Flop. It also has an AND and OR gate.
- We then convert it into squares since we need appropriate dimensions

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/f7148664-ea48-420e-92a6-b5ef5ebc30fd)
- Let us consider the areas of the gates and Flops as 1 sq unit

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/6b3f5692-c22e-4c21-857d-d689adf834f0)
- Clubbing them together we get an area of 4 sq units

- The 'core' section of a chip is where the fundamental logic design is placed.
- The 'die' area contains the core and is a small semiconductor are on which the fundamental circuit is fabricated.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/b2c4dbb6-2b0c-477c-9696-a5fe90c282ef)
- Now we put the netlist in the 'core' area and check the utilization.
- Here
```
Utilization Factor = Area Occupied by the Netlist/Total Area of the Core
```
- As we can see here, there is 100% utilization and ```Utilization Factor = 1```.
- In practical scenarios we don't go for such a high utilization factor.
- The 'Aspect Ratio = Height/Width = 1'.

**Concept of Pre Placed Cells**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/ce3ce1b6-578f-4910-924c-d712f174e809)
- We take the above combinational logic as an example

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/a9cd2668-26b2-41ec-9c04-cb5b1d85f2a5)
- We split the circuit into two parts, block 1 and block 2 as shown above

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/6a468eae-b0f1-4eb7-ad74-0ba8ba42626d)
- We extend the I/O pins and black box the boxes.
- Now we separate the boxes and the get their respective I/O ports.
- The use of doing this is that the users can use the blocks multiple times and form the required final circuit with ease.
- They only need to implement the design once and it can be reused.
- These kind of IPs have user defined locations and are placed in the chip before automated placement and routing takes place. These are called pre-placed cells.

**Surrounding Pre-Placed Cells with Decoupling Capacitors**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/10b42d05-a8a2-4a44-bd0c-08102b608e38)

- Huge capacitor filled with charge. The equivalent voltage across the capacitor is similar to what the power supply produces.
- We add the capacitor in parallel to the circuit.
- Everytime the circuit switches it draws current from the decoupling capacitor, whereas the outer network with the power supply and other componets is used to re-charge the capacitor

**Pin Placement**
- In pin placemnt step we use the HDL netlist to determine where a specific pin should be placed in the circuit.
- We join the common pins and try to keep the connections as effecient as possible.
- Pins are placed in the Die area.

**Steps to run FLoorplan using OpenLANE**
- To view floorplan we type
```
run_floorplan
```

- To open the Floorplan we go to the required directory that is
```
vsduser@vsdsquadron:~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/11-09_15-36/results/floorplan
```
using the ```cd``` command.

- Then we type the command:
```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

- The following layout is displayed
![Screenshot from 2023-09-17 12-52-29](https://github.com/PoojaR07/pes_pd/assets/135737910/3b9ac67b-81ca-4a93-92a3-d092100ecafb)

- We can right click on the mouse and pess 'z' to zoom into a desired part.
![Screenshot from 2023-09-17 13-05-37](https://github.com/PoojaR07/pes_pd/assets/135737910/87eae289-f042-4e21-b85c-afe193c6bf5c)

- We can see here that the I/O ports are equidistant
![Screenshot from 2023-09-17 13-06-20](https://github.com/PoojaR07/pes_pd/assets/135737910/1a8f36f7-470f-4c21-b290-5b0d6665b742)

- Standard cells that are used in the design
![Screenshot from 2023-09-17 13-08-54](https://github.com/PoojaR07/pes_pd/assets/135737910/accd691d-c0b8-4229-931d-de6d370819b9)

## Library Binding and Placement

**Netlist Binding and Initial Place Design**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/9fbd1dd6-34b3-4b38-b92a-c56efe08311f)
- In real life, the logic gates and cells do not have shapes, but are present in the form of rectangles and squares.
- Hence they have dimensions to them and the space where they are placed must be utilized carefully
- The above picture shows an example of a library.
- Library consists of various kinds of cells which have different shapes and sizes, flavours and different timing information.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/adc000d3-4076-4c74-b6d3-96e3e10f311f)
- The components of the netlist are placed in the core area.
- They are placed according to the convenience of distance from the pins.
- When sending signal from FF1 to FF2, according to the circuit requirements, there has to be a very fast propogation of signals. Hence, they are placed very close and buffers are added since there is a small delay for the signal from the pin to reach FF1. The buffers maintain signal integrity

**Viewing the Placement**
- To view the placement we type
```
run_placement
```
in the OpenLANE shell.

![Screenshot from 2023-09-17 13-19-36](https://github.com/PoojaR07/pes_pd/assets/135737910/8ce41ee5-c6b3-461a-b4fa-31517ee25c3f)

- We move one directory up from the 'floorplan' folder using
```
cd ../placement/
```
- To view the placement design we use the command
```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def
```
![Screenshot from 2023-09-17 13-22-55](https://github.com/PoojaR07/pes_pd/assets/135737910/72bb3a59-dfeb-442c-9227-64a3509757ca)

- The above is displayed.
- All these standard cells were present at the initial layout of the floorplan.
![Screenshot from 2023-09-17 13-24-15](https://github.com/PoojaR07/pes_pd/assets/135737910/8493d357-88d7-4cb0-871d-17bd2c6d695c)

- If we zoom in we can see the placement of the standard cells in the standard cell rows.

## Cell Design and Characterization Flow

**Cell Design Flow**
- Inputs -> Process design kits(PDKs) : DRC and LVS rules, SPICE models, library and user-defined specs.
- Design Steps -> Circuit Design, Layout Design(Euler Path and Stick Diagram), Characterization.
- Outputs -> CDL(Circuit Description Language), GDSII, LEF, extracted spice netlist(.cir)

**Characterization Flow**
- This is for an inverter.
1) Read the model files.
2) Read the extracted SPICE netlist.
3) Recognize the behaviour of the buffer.
4) Attaching the necessary power sources
5) Apply the stimulus, which is the input signal to the circuit.
6) Read the sub-circuit of the inverter.
7) Provide necessary output capacitances.
8) Provide the necessary simulation commands

**Timing Characterization**
- slew_low_rise_thr = 20%
- slew_high_rise_thr = 80%
- slew_low_fall_thr = 20%
- slew_high_fall_thr = 80%
- in_rise_thr = 50%
- in_fall_thr = 50%
- out_rise_thr = 50%
- out_fall_thr = 50%

- Propogation delay = time(out_fall_thr) - time(in_rise_thr)

- Transition Time
  - On rise: time(slew_high_rise_thr) - time(slew_low_rise_thr)
  - On fall : time(slew_high_fall_thr) - time(slew_low_fall_thr)
</details>

<details>
<summary>DAY 3 : Design library cell </summary>

## Labs for CMOS inverter ngspice simulations
**IO Placer Revision**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/fabe5ca4-7dde-43f9-a8e4-260ed11ed820)
- The following command can be typed to change the I/O pins placemnt configuration.

## Inception of Layout and CMOS Fabrication Process
**SPICE Deck Creation for CMOS Inverter**
- SPICE Deck is a netlist that has information on:
  - component connectivity 
  - component values
  - identifying the nodes
  - giving a designation to the nodes

**SPICE Simulation and Switching Threshold**

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/29e6f5c4-d166-4283-85c2-81947d29f165)
- The CMOS on the right side has a bigger size than the one on the left.
- These waveforms tell us that the CMOS is a very robust device. The characteristics of the CMOS are maintained across a variety of sizes.
- The arrow is pointing to the point where 'Vin = Vout'.

![image](https://github.com/AniruddhaN2203/pes_pd/assets/142299140/247e37b7-b3b3-4036-9eaf-2c5380a6c71a)
- Above graph gives details on each point and its significance

**A Git Clone and some other Steps**

- We need to perform a git clone here from a repository that we require, to do the future labs.
- We can type the following command
```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

- Now we need to copy the 'sky130A.tech' file into the directory we just cloned
- We can do this by using
```
cp sky130A.tech /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
```
in the follwoing directory shown in the figure
![Screenshot from 2023-09-17 19-01-22](https://github.com/PoojaR07/pes_pd/assets/135737910/43c2b28a-99b2-4d37-ae07-378d4d279923)


**16 Mask CMOS Process**
1) Selecting a Substrate - Selecting the appropriate substrate to synthsize the design on.
2) Creating active reagion for transistors - Adding layers of SiO2(40nm), Si3N4(80nm) and photoresist(1um). On top of the photoresist we put a mask layer. Pass UV light and remove the mask. Resist is removed. LOCOS(Local Oxidation of Silicon) is performed. Si3N4 is etched.
3) N-Well and P-Well formation - The next masks are used to create the source and drain regions of the MOSFETs. Boron is used to make P-Well using ion implantation. Phosphorus is used to create N-Well. Put the MOSFET in a Drive In furnace.
4) Formation of Gate - Gate formation involves depositing a gate oxide, defining gate patterns using photolithography, depositing gate material, etching to create gates, doping the substrate and insulating the gates.
5) Lightly Doped Drain Formation(LDD) - Lightly doped drain (LDD) formation involves implanting the drain and source regions of a MOSFET transistor with a lighter concentration of dopants to reduce hot electron effect and short channel effect and enhance device performance.
6) Source and Drain Formation - Source and drain formation in a MOSFET transistor typically involves doping the silicon substrate with chemicals such as arsenic or phosphorous for n-type regions (source and drain) and boron for p-type regions (source and drain). High temperature annealing is performed.
7) Steps to form Contacts and Interconnects(local) - Titanium is deposited with a process known as sputtering. Wafer is heated to about 650 - 700 C in an N2 ambient furnace for 60 seconds. TiSi2 contacts are formed.  TiN is also formed used for local communication. TiN is etched using RCA cleaning.
8) Higher Level Metal Formation - Forming contacts and interconnects locally involves depositing a dielectric material like silicon dioxide, patterning it using photolithography, etching contact holes, depositing a barrier metal (e.g., titanium or titanium nitride), filling with a conductor (e.g., aluminum or copper) using chemical vapor deposition (CVD), and then planarizing through chemical-mechanical polishing (CMP).

**Sky130 Basic Layers Layout and LEF using Inverter**
- Now let us look at the layout of a CMOS inverter. To open this we type the command
![Screenshot from 2023-09-17 19-03-03](https://github.com/PoojaR07/pes_pd/assets/135737910/12a36370-265d-49dd-a828-e8baaf5895e7)


```
 magic -T sky130A.tech sky130_inv.mag &
```
![Screenshot from 2023-09-17 19-03-28](https://github.com/PoojaR07/pes_pd/assets/135737910/c943b6b4-2b69-46eb-87ea-c882f4524981)

- The following layout is displayed.
  
![Screenshot from 2023-09-17 19-13-28](https://github.com/PoojaR07/pes_pd/assets/135737910/b01a676c-3d96-4640-8b96-2c77599a7864)

**Extracting PEX to SPICE with MAGIC**
- To extract Spice Netlist we perform the following steps in the tkcon window:
![Screenshot from 2023-09-17 19-24-36](https://github.com/PoojaR07/pes_pd/assets/135737910/8def0ad4-52dc-45dc-a3c7-b24b5bb5dd0e)

![Screenshot from 2023-09-17 19-32-36](https://github.com/PoojaR07/pes_pd/assets/135737910/9ba69963-e4fa-4aa7-b2f5-d9a2bc84fa69)

![Screenshot from 2023-09-17 19-31-52](https://github.com/PoojaR07/pes_pd/assets/135737910/8ce809da-46f7-4bb5-8d0f-bce5a2de8e05)

- The above file has details of inverter netlist but the sources and their values are not specified. So we have to modify the file.
    - Grid size from the layout is 0.01u
    - specify the library for MOS
    - create VDD, VSS, Input pulse Va
    - specify the type of analysis to be done

- Grid Size
![Screenshot from 2023-09-17 21-03-46](https://github.com/PoojaR07/pes_pd/assets/135737910/c4e72285-c096-462b-9730-a69d844e121f)

- Modified Spice netlist
![Screenshot from 2023-09-17 22-14-33](https://github.com/PoojaR07/pes_pd/assets/135737910/ef6e4d50-8e99-42c8-bf4e-df397abe11f9)

**NGPSICE**

![Screenshot from 2023-09-17 22-13-29](https://github.com/PoojaR07/pes_pd/assets/135737910/7ccd142a-9c4c-4ecc-a7b0-3d432abb6092)

- In the ngspice shell we use the command
```
plot y vs time a
```
![Screenshot from 2023-09-17 22-40-09](https://github.com/PoojaR07/pes_pd/assets/135737910/0212760f-2018-4db0-96a8-adffd38377a4)
- The following graph is displayed

![Screenshot from 2023-09-18 00-19-23](https://github.com/PoojaR07/pes_pd/assets/135737910/9edf0253-bcee-43df-85d7-bc8c93282599)

![Screenshot from 2023-09-18 00-20-13](https://github.com/PoojaR07/pes_pd/assets/135737910/ce0a47ce-3111-4c9d-b9d7-2eb79438af9a)

- Rise Time -> time taken to rise from 20% to 80% of the max value -> 2.25075e-09 - 2.184e-09 = 0.0412e-09 s.

![Screenshot from 2023-09-18 00-28-44](https://github.com/PoojaR07/pes_pd/assets/135737910/4997bc2f-53e6-4fba-9898-ba595635b329)

![Screenshot from 2023-09-18 00-28-27](https://github.com/PoojaR07/pes_pd/assets/135737910/512ad5dd-dfc8-4f63-b95e-5f606cccab31)

- Propogation Delay/Cell Rise Delay -> 2.21379e-09 - 2.15e-09 = 0.03604e-09 s.

**Introduction to Magic tool options and DRC rules**
- Magic is a venerable VLSI layout tool, written in the 1980's at Berkeley by John Ousterhout, now famous primarily for writing the scripting interpreter language Tcl. Due largely in part to its liberal Berkeley open-source license, magic has remained popular with universities and small companies. The open-source license has allowed VLSI engineers with a bent toward programming to implement clever ideas and help magic stay abreast of fabrication technology. However, it is the well thought-out core algorithms which lend to magic the greatest part of its popularity. Magic is widely cited as being the easiest tool to use for circuit layout, even for people who ultimately rely on commercial tools for their product design flow.

- Drc section The design rules used by Magic's design rule checker come entirely from the technology file. We'll look first at two simple kinds of rules, width and and spacing. Most of the rules in the drc section are one or the other of these kinds of rules.

- SKY130 pdk SKY130 is a mature 180nm-130nm hybrid technology developed by Cypress Semiconductor that has been used for many production parts. SKY130 is now available as a foundry technology through SkyWater Technology Foundry.

**Sky130 PDKS and Steps to Download Magic Tool**
![Screenshot from 2023-09-18 01-01-42](https://github.com/PoojaR07/pes_pd/assets/135737910/13ec6bbd-8e14-421d-88e2-7d0ea2413361)

![Screenshot from 2023-09-18 01-04-53](https://github.com/PoojaR07/pes_pd/assets/135737910/7404e257-2dad-4563-b7c1-b2ef5f703f72)


- To open the software we type
```
magic -d XR
```
![Screenshot from 2023-09-18 01-08-03](https://github.com/PoojaR07/pes_pd/assets/135737910/b95dc28a-ba27-4a60-94ac-7bb1a9d44258)

- To check which DRC rule is being violated select area and type drc why in tkcon
![Screenshot from 2023-09-18 01-35-04](https://github.com/PoojaR07/pes_pd/assets/135737910/4305a6b8-bd8f-4f1d-9f0a-f88f1d52e65a)
  
- To add contact cuts to metal3, first select an area using left and right click. Then hovering over the m3contact we click middle mouse button.

**Fixing DRC Errors**
- There is a DRC error in the poly.mag file in 'poly.9'.
- Open the sky130A.tech file in the editor and make the following changes

![Screenshot from 2023-09-18 15-09-47](https://github.com/PoojaR07/pes_pd/assets/135737910/fa537812-05a0-4adf-a264-53d99ef90988)

![Screenshot from 2023-09-18 15-10-19](https://github.com/PoojaR07/pes_pd/assets/135737910/0df14178-9c6d-437e-a5d0-2e16dbec8d3f)

- Now load the sky130A.tech file again and type the command drc check

![Screenshot from 2023-09-18 15-08-54](https://github.com/PoojaR07/pes_pd/assets/135737910/377ef9bf-9f28-446f-80a5-880e68a0bf2e)

- We can see the error is fixed
![Screenshot from 2023-09-18 15-09-05](https://github.com/PoojaR07/pes_pd/assets/135737910/33f963a3-e1d5-49e3-a613-94f3784b5240)

</details>
