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

![Screenshot 2023-09-19 114203](https://github.com/PoojaR07/pes_pd/assets/135737910/eebb382b-d84e-489f-a70a-1efb36e1be19)

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

### Lab challenge exercise to describe DRC error as geometrical construct:

* Open the nwell.mag file.
  
![Screenshot 2023-09-19 114318](https://github.com/PoojaR07/pes_pd/assets/135737910/e0954bfc-3fff-45f8-922f-6c469a5736df)

* Type the following commands:
  + `cif ostyle drc`
  + `cif see dnwell_shrink`
  + `cif see nwell_missing`
  
![Screenshot 2023-09-19 114410](https://github.com/PoojaR07/pes_pd/assets/135737910/2ffd3c0a-2c28-4159-ac4c-a0a064f78b08)

### Lab challenge to find missing or incorrect rules and fix them:

![Screenshot 2023-09-19 114445](https://github.com/PoojaR07/pes_pd/assets/135737910/9dda1863-c729-41c5-89a0-88d1115aa49a)

* Make the following changes:
  
![Screenshot 2023-09-19 114534](https://github.com/PoojaR07/pes_pd/assets/135737910/cf385c56-5141-42f3-8ba1-afaa2a255f55)

* Run the below commands:

![Screenshot 2023-09-19 114623](https://github.com/PoojaR07/pes_pd/assets/135737910/136ea4c2-5bea-42b1-ad28-d0a0b61066e1)

* We observe that the error is still seen.
  
![Screenshot 2023-09-19 114653](https://github.com/PoojaR07/pes_pd/assets/135737910/76795c39-8059-4a6c-838c-ec22c95868e6)

* To correct this error:
  + Select the nwell.4
  + Make a copy of it.
  + Now select a small area on the nwell.4 and add an 'nsubstratecontact'.
  
![Screenshot 2023-09-19 114721](https://github.com/PoojaR07/pes_pd/assets/135737910/dba3159c-aecc-44f3-a827-3eb9902993f0)

</details>

<details>
<summary>DAY 4 : Pre-Layout timing analysis and importance of good clock tree</summary>

## Extraction of LEF 

Place and routing (PnR) is performed using an abstract view of the GDS files generated by Magic. The abstract information will include metal and pin information. The PnR tool will use the abstract view information, formally defined as LEF information, to perform interconnect routing in conjunction to routing guides generated from the PnR flow.

- Technology LEF - Contains layer information, via information, and restricted DRC rules
- Cell LEF - Abstract information of standard cells

From PnR POV, We have to follow certain guidelines to get standard cell set
1. Input and output ports must lie on the intersection of vertical and horizontal tracks
2. Width of the standard cell should be odd multiples of the track pitch and height should be odd multiple of vertical track pitch


Track info can be found at :

``` ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130fd_sc_hd/tracks.info```

![Screenshot from 2023-09-18 16-25-41](https://github.com/PoojaR07/pes_pd/assets/135737910/21623b37-8303-4ffe-a93b-c2da97a4c236)

- 1st value indicates the offset and 2nd value indicates the pitch along provided direction

### Setting grid values using above file info

![Screenshot from 2023-09-18 16-29-48](https://github.com/PoojaR07/pes_pd/assets/135737910/96349fb3-9148-49f7-8b2f-06a8ac359939)

Layout after setting grid info

![Screenshot from 2023-09-18 16-29-59](https://github.com/PoojaR07/pes_pd/assets/135737910/dc5a5cb7-2b5c-42c3-aa46-95941191f9c6)

![Screenshot from 2023-09-18 16-30-45](https://github.com/PoojaR07/pes_pd/assets/135737910/78ca9de5-1269-4b86-8d1a-80799c143686)


- From the above pic, its confirmed that the pins A and Y are at the intersection of X and Y tracks. So the first condition is met.
- The PR boundary is taking 3 grids on width and 9 grids on height which says that the 2nd condition is also met

## LEF Generation

Since the layout is perfect, we can generate the lef file

#### 1. save the modified layout (with new grid)
   - In console, type ```save sky130_vsdinv.mag```
   - This saves the modified layout in current working directory

#### 2. Open the file and extract LEF
   - Open using ``` magic -T sky130A.tch sky130_vsdinv.mag```
   - in the console opened, type ```lef write``` and a lef file will be generated
![Screenshot from 2023-09-18 16-49-19](https://github.com/PoojaR07/pes_pd/assets/135737910/7342edb1-1977-4b3e-88c7-278871cc4ef5)

#### 3. Plug the generated lef file into PICORV32a

To do this, we need the lef file, library file that has cells

![Screenshot from 2023-09-18 16-55-19](https://github.com/PoojaR07/pes_pd/assets/135737910/98cb3e86-0ad5-4f21-9668-52cc3f937e09)

Change config file so that these libraries and lef file is used

![Screenshot from 2023-09-18 18-03-36](https://github.com/PoojaR07/pes_pd/assets/135737910/d5de251c-2317-4978-9a6d-4566124bac78)

#### 4. Make sure the lef file is added

Next in OpenLANE we retrieve the 0.9 package.

We type the followig commands 
```
prep -design picorv32a -tag 18-09_05-15 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```

and ```run_synthesis```  to see if our inverter has been used and find timing violations if any.
![Screenshot from 2023-09-18 18-20-55](https://github.com/PoojaR07/pes_pd/assets/135737910/9dd4446f-751e-426f-83e1-b9403795cd6e)

![Screenshot from 2023-09-18 18-31-48](https://github.com/PoojaR07/pes_pd/assets/135737910/80945034-741f-4b2a-9e68-a763f07103c6)



**steps to configure synthesis settings to fix slack and include vsdinv**

Slack violations refer to timing violations in digital designs. These violations occur when the signal arrives at its destination too early or too late, violating the specified setup or hold time constraints. 

When referring to pre clock tree synthesis STA analysis we are mainly concerned with setup timing in regards to a launch clock. STA will report problems such as worst negative slack (WNS) and total negative slack (TNS). These refer to the worst path delay and total path delay in regards to our setup timing restraint. Fixing slack violations can be debugged through performing STA analysis with OpenSTA, which is integrated in the OpenLANE tool.
The desired value of slack is above or equal to 0. 

Ways to fix slack

1.Changing  synthesis strategy in OpenLANE
 - Enalbed CELL_SIZING
 - Enabled SYNTH_STRATEGY with parameter as DELAY 1

The delay is high when the fanout is high we can re-run synthesiwith different values of SYNTH_MAX_FANOUT variable

2.Enable cell buffering

3.Perform manual cell replacement on our WNS path with the OpenSTA tool

4.Optimize the fanout value with OpenLANE tool

Now since synthesised the core using our vsdinv cell too and as it got successfully synthesized. We go ahead with the floorplan 

```
init_floorplan
run_placement
```
![Screenshot from 2023-09-18 19-07-54](https://github.com/PoojaR07/pes_pd/assets/135737910/2990a265-95b8-44a0-8685-e3405ce71593)


On zooming in 

![Screenshot from 2023-09-18 19-14-09](https://github.com/PoojaR07/pes_pd/assets/135737910/b88b699a-032b-4a73-a082-81741defa6f9)



**Introduction to delay tables**

Delay tables, also known as delay models or delay tables, are essential components in digital circuit design and analysis. They provide a way to model and understand the propagation delays of logic gates and interconnects within a digital integrated circuit (IC). These tables play a crucial role in ensuring that the circuit meets its timing requirements, such as setup and hold times, and they are fundamental to the design of synchronous digital systems. Here's an introduction to delay tables:

1. Purpose of Delay Tables:

Delay tables are used to represent the delays encountered by signals as they pass through various components of a digital circuit. The primary purposes of delay tables are as follows:

Timing Analysis: They are essential for performing timing analysis, ensuring that signals meet their timing constraints, and identifying potential violations.

Synchronization: They help in synchronizing different parts of a digital system to ensure that data is sampled or latched correctly.

Power Estimation: Delay tables are used for estimating power consumption in digital circuits since power dissipation is directly related to signal transitions.

2. Components of Delay Tables:

Delay tables typically include the following components:

Input Conditions: These conditions specify the input signal values or transitions that trigger the delay calculation. Inputs can include input signal values, load conditions, and transition times.

Gate Delays: Delay tables include information about the propagation delays of various logic gates, such as AND, OR, NAND, NOR, XOR, and others. These delays depend on the gate's technology, fan-out, and input conditions.

Interconnect Delays: They account for the delays introduced by the wires and routing between logic gates. Interconnect delays depend on the physical characteristics of the wires, including length, resistance, and capacitance.

Output Loads: The output load conditions specify the capacitive load that the gate must drive, which affects the output delay.

</details>

<details>
<summary>DAY 5 : Final steps for RTL2GDSII</summary>

## Power Distribution Network and Routing

PDN (Power Delivery Network) routing is a crucial aspect of integrated circuit design. It involves the creation of a network of traces and components to ensure that power is distributed effectively and reliably to all parts of the electronic device. 

Global and detailed routing are two essential steps in the design and manufacturing of integrated circuits. 
After generating our clock tree network  we  generate the power distribution network gen_pdn using  OpenLANE:

The PDN  will create:

- Power ring global for the entire core
A global power ring is a continuous metal ring that surrounds the entire core of the IC.It's used to distribute power (VDD) uniformly to the core logic and various functional blocks.The power ring ensures that all regions of the core receive power without significant voltage drops.

- Power halo local to any preplaced cells
A power halo is a localized power distribution network around specific preplaced cells or macroblocks on the chip.Preplaced cells are often fixed in their positions, and a power halo provides them with the necessary power connections.

- Power straps to bring power into the center of the chip
Power straps are metal traces or structures used to bring power from the periphery of the chip towards the central regions.They are essential for delivering power to the core logic and other critical areas, reducing the distance power must travel.Power straps help maintain uniform power distribution across the chip.

- Power rails for the standard cells
Power rails are metal lines that run vertically or horizontally across the chip, supplying power to standard cells .These power rails ensure that each standard cell has access to the power it needs for proper operation.

```gen_pdn```

![Screenshot 2023-09-19 115709](https://github.com/PoojaR07/pes_pd/assets/135737910/f4483964-9f60-4dd9-9b05-f37ac7e13415)

![Screenshot 2023-09-19 115747](https://github.com/PoojaR07/pes_pd/assets/135737910/440435c1-c7da-4aeb-b44d-86a4ae34db96)


to run the rounting we type ```run_routing```

## SPEF Extraction

SPEF stands for Standard Parasitic Exchange Format, and it is a standard file format used in the semiconductor industry to represent parasitic information for integrated circuits. Parasitic elements, such as resistance and capacitance, can significantly affect the performance of a circuit, so accurate modeling and extraction of these parasitics are crucial for designing and optimizing electronic devices.After routing has been completed interconnect parasitics can be extracted into a SPEF file. The SPEF extractor is nota part of OpenLANE as of now.

Commands is 
```
cd Desktop/work/tools/SPEF_Extractor
```

then we type 
```
python3 /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/18-09_06-26/tmp/merged.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/18-09_06-26/results/routing/picorv32a.def
```

The SPEF exracted file is created. 
Path to the created files is 
```
/home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/18-09_06-26/results/routing/
```
</details>
