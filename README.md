# Advanced Physical Design Using OPENLANE/SKY130
<details>
  <summary>Prerequisites</summary>
  
 1. Ubuntu OS-based System
  
 2. 25GB+ Disk Space
      
 Please refer to: https://github.com/nickson-jose/openlane_build_script for installation steps
</details>
<details>
<summary>RTL to GDSII Introduction</summary>

The ASIC design flow is a dynamic and iterative process, subject to variations based on factors like Engineering Change Orders (ECOs), Intellectual Property (IP) requirements, Design for Test (DFT) insertion, and Synchronous Design Constraints (SDC). Despite these potential modifications, the fundamental concepts of the flow can be broken down into 11 steps:

1. Architectural Design: This step begins with system engineers providing VLSI engineers with specifications based on physical constraints. VLSI engineers design a circuit at a microarchitecture modeling level that meets these requirements.

2. RTL Design/Behavioral Modeling: Using a Hardware Description Language (HDL) like Verilog or VHDL, this phase involves creating an abstraction of the digital circuit, representing it with combinational logic, registers, and modules (such as IP blocks or Soft Macros).

3. RTL Verification: Behavioral verification of the RTL design is carried out to ensure that it meets the desired functionality and requirements.

4. DFT Insertion: Design-for-Test circuitry is inserted into the design to facilitate testing and verification of the ASIC.

5. Logic Synthesis: This process transforms the RTL netlist into a lower-level representation, typically involving two key steps: GTECH mapping and technology mapping. GTECH mapping maps the HDL netlist to generic gates for logical optimization, while technology mapping maps the optimized netlist to standard cells from the Process Design Kit (PDK).

6. Standard Cells: Standard cells are fixed-height and have a width that is a multiple of the SITE size or PR boundary. They come with various files used by different tools throughout the design flow.

7. Post-Synthesis STA Analysis: Setup analysis is performed on different path groups to ensure proper timing constraints are met.

8. Floorplanning: The goal here is to plan the silicon area and establish a robust power distribution network (PDN) while defining macro placement and blockages to generate a legal GDS (Graphic Design System) file.

9. Placement: Standard cells are placed on the floorplan rows, aligned with predefined sites in the technology LEF file. This process occurs in two steps: global placement, which finds initial positions for cells, and detailed placement, which refines these positions to meet global placement guidelines.

10. CTS (Clock Tree Synthesis): CTS is used to create a clock distribution network that minimizes clock skew across the chip. Common network topologies like H-trees are employed for this purpose.

11. Routing: The interconnect system between standard cells is implemented using available metal layers after CTS and PDN generation. Routing is conducted on routing grids to minimize Design Rule Check (DRC) errors.

The Skywater 130nm Process Design Kit (PDK) utilizes six metal layers for various purposes, including CTS, PDN generation, and interconnect routing. These layers play a crucial role in achieving optimal performance and reliability in the ASIC design.

Shown below is an example of a base RTL to GDS flow in ASIC design:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/1de4339c-7d98-41ff-812b-b7531bf73d64)

</details>

<!-- Workshop Introduction -->
## Workshop Introduction

The inputs to the ASIC design flow encompass critical components such as Process Design Rules (DRC, LVS, PEX), Device Models (SPICE), Digital Standard Cell Libraries, and I/O Libraries. These foundational elements serve as the building blocks for the design process.

The Process Design Kit (PDK) acts as the crucial interface between CAD designers and foundries. It comprises a collection of files that effectively model the fabrication process for Electronic Design Automation (EDA) tools employed in IC design. Traditionally, PDKs have been proprietary and closed-source, posing limitations for open-source Digital ASIC Design initiatives. However, Google and Skywater made a groundbreaking contribution by releasing the world's first open-source PDK on June 30th, 2020. This monumental step has ushered in a new era of open-source EDA tools and has become a driving force for democratizing the ASIC design process.

The focus of this workshop is to leverage the power of the open-source RTL2GDS EDA tool, OpenLANE, in conjunction with the Skywater 130nm PDK. This combination enables engineers and designers to execute the complete RTL2GDS flow, thereby taking advantage of the openness and accessibility offered by the Skywater PDK to realize their ASIC design objectives.

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/b6505bf2-e179-4d8c-b076-a08ca84a3cb7)

OpenLANE flow consists of several stages. By default, all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLANE can also be run interactively as shown here.

  1. Synthesis

  <ul>
      <li>Yosys - Performs RTL synthesis using GTech mapping</li>
      <li>abc - Performs technology mappin to standard cells described in the PDK. We can adjust synthesis techniques using different integrated abc scripts to get desired results</li>
      <li>OpenSTA - Performs static timing analysis on the resulting netlist to generate timing reports</li>
      <li>Fault – Scan-chain insertion used for testing post fabrication. Supports ATPG and test patterns compaction</li>
  </ul>

  2. Floorplan and PDN

  <ul>
      <li>Init_fp - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)</li>
      <li>Ioplacer - Places the macro input and output ports</li>
      <li>PDN - Generates the power distribution network</li>
      <li>Tapcell - Inserts welltap and decap cells in the floorplan</li>
      <li>Placement – Placement is done in two steps, one with global placement in which we place the designs across the chip, but they will not be legal placement with some standard cells overlapping each other, to fix this we perform a detailed placement which legalizes the design and ensures they fit in the standard cell rows</li>
      <li>RePLace - Performs global placement</li>
      <li>Resizer - Performs optional optimizations on the design</li>
      <li>OpenPhySyn - Performs timing optimizations on the design</li>
      <li>OpenDP - Perfroms detailed placement to legalize the globally placed components</li>
  </ul>
  3. CTS

  <ul>
      <li>TritonCTS - Synthesizes the clock distribution network</li>
  </ul>


4. Routing

  <ul>
      <li>FastRoute - Performs global routing to generate a guide file for the detailed router
      </li>
      <li>TritonRoute - Performs detailed routing from global routing guides</li>
      <li>SPEF-Extractor - Performs SPEF extraction that include parasitic information</li>
  </ul>

5. GDSII Generation

  <ul>
      <li>Magic - Streams out the final GDSII layout file from the routed def</li>
  </ul>

 6. Checks

  <ul>
      <li>Magic - Performs DRC Checks & Antenna Checks</li>
      <li>Netgen - Performs LVS Checks </li>
  </ul>
  
## Workshop Contents
<details>
<summary>DAY 1--Inception of Open Source EDA</summary>
  
## Skywater PDK Files

The Skywater PDK files we are working with are described under $PDK_ROOT. There are three subdirectories needed for the workshop:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/8fa0d00f-e8d3-403c-9fa7-0aab90889531)

  1. Skywater-pdk – Contains all the foundry provided PDK related files
  2. Open_pdks – Contains scripts that are used to bridge the gap between closed-source and open-source PDK to EDA tool compatibility
  3. Sky130A – The open-source compatible PDK files

## Invoking OpenLane

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/6f1388c3-b14d-4fe3-9e1f-1ae424b9822d)

  - ./flow.tcl is the script which runs the OpenLANE flow
  - OpenLANE can be run interactively or in autonomous mode 
  - To run interactively, use the -interactive option with the ./flow.tcl script 

## Package Importing
Different software dependencies are needed to run OpenLANE. To import these into the OpenLANE tool we need to run:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/cd491706-c464-467d-bf5c-1bdfc4d9bf1e)


## Design Folder
All designs run within OpenLANE are extracted from the openlane/designs folder:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/27b928d7-a035-4c42-9334-7379c9fe15d3)


## Design Folder Hierarchy

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/f74dac58-fb93-4c26-a270-be6d664764ef)

Each design hierarchy comes with two distinct components:
  1. Src folder - Contains verilog files and sdc constraint files
  2. Config.tcl files - Design specific configuration switches used by OpenLANE

An example of a configuration file is given:

 ![image](https://github.com/spurthimalode/pes_pd/assets/142222859/cfd77f38-dcae-40a1-b5b1-311d1934e924)


## Prepare Design
Prep is used to make file structure for our design. To set this up do:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/5687e1a3-fbf0-4979-821d-aee2a12710b5)


After running this look in the openlane/design/picro32a folder and you will see there is a new directory structure created in this folder under the runs folder so to enable OpenLANE flow:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/af4d2d2f-e789-410c-b9e3-2c75ca1bc395)


## Synthesis

To run synthesis: Use `run_synthesis` command

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/0cc1bc84-0367-42a7-a6a9-9179f89b0b64)

</details>

<details>
<!-- Day 2 Chip Floorplanning and Standard Cells-->
<summary>DAY 2 -- Chip Floorplanning and Standard Cells</summary>

In Floorplanning we typically set the:
  1. 	Die Area
  2. 	Core Area
  3. 	Core Utilization
  4. 	Aspect Ratio
  5. 	Place Macros
  6. 	Power distribution network (Normally done here but done later in OpenLANE)
  7. 	Place input and output pins

### Aspect Ratio and Utilization Factor

In ASIC design, two crucial parameters that define the floorplan of the chip are utilization and aspect ratio. Utilization refers to the proportion of the die core occupied by standard cells, typically targeted within the range of 50-70% or a utilization factor of 0.5-0.7. This range ensures an optimal balance for placement and facilitates feasible routing within the system. Aspect ratio, on the other hand, characterizes the chip's shape by dividing the height of the core area by its width. An aspect ratio of 1 signifies a square-shaped chip, while other ratios define rectangular configurations. These two descriptors play a pivotal role in shaping the physical layout of an integrated circuit.

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/829699f8-6843-416c-b692-5985470a7793)

### Preplaced Cells

Preplaced cells, or MACRO’s, are important to enable hierarchical PnR flow. Preplaced cells enable VLSI engineers to granularize a larger design. In floorplanning we define locations and blockages for preplaced cells. Blockages are needed to ensure no standard cells are mapped where the placeplaced cells are located.

### Decoupling Capacitors

Decoupling capacitors are placed local to preplaced cells during Floorplanning. Voltage drops associated with interconnect wires can heavily affect our noise margin or put it into an indeterminate state. Decoupling capacitor is a big capacitor located next to the macros to fix this problem. The capacitor will charge up to the power supply voltage over time and it will work as a charge reservoir when a transition is needed by the circuit instead of the charge coming from the power supply. Therefore it “decouples” the circuit from the main supply. The capacitor acts like the power supply.

### Power Planning

Power planning during the Floorplanning phase is essential to lower noise in digital circuits attributed to voltage droop and ground bounce. Coupling capacitance is formed between interconnect wires and the substrate which needs to be charged or discharged to represent either logic 1 or logic 0. When a transition occurs on a net, charge associated with coupling capacitors may be dumped to ground. If there are not enough ground taps charge will accumulate at the tap and the ground line will act like a large resistor, raising the ground voltage and lowering our noise margin. To bypass this problem a robust PDN with many power strap taps are needed to lower the resistance associated with the PDN.

### Pin Placement

Pin placement is an essential part of floorplanning to minimize buffering and improve power consumption and timing delays. The goal of pin placement is to use the connectivity information of the HDL netlist to determine where along the I/O ring a specific pin should be placed. In many cases, optimal pin placement will be accompanied with less buffering and therefore less power consumption. After pin placement is formed we need to place logical cell blockages along the I/O ring to discriminate between the core area and I/O area.

### Floorplanning with OpenLANE

To run floorplan in OpenLANE: 
```
cd OpenLane
sudo make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design <file_name>
run_synthesis
run_floorplan
```
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/9ccb2e3f-5737-4bc3-b50f-c61c0337b3c5)

config.tcl 
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/d7943468-a392-4a17-9a6a-0f4202efc0c8)

As with all other stages, the floorplanning will be run according to configuration settings in the design specific config.tcl file. The output the the floorplanning phase is a DEF file which describes core area and placement of standard cell SITES:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/2b65a4eb-4ace-4d1b-8fa1-8b71e97840a0)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/fac06ff3-bdbe-4f57-80bf-de3083531316)


### Viewing Floorplan in Magic
To view our floorplan in Magic we need to provide three files as input:

  1. Magic technology file (sky130A.tech)
  2. Def file of floorplan
  3. Merged LEF file

```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```
 ![image](https://github.com/spurthimalode/pes_pd/assets/142222859/5624cf46-c267-4474-87bf-1cc1f2f35365)

### Placement

The next step in the Digital ASIC design flow after floorplanning is placement. The synthesized netlist has been mapped to standard cells and floorplanning phase has determined the standard cells rows, enabling placement. OpenLANE does placement in two stages:

  1. Global Placement - Optimized but not legal placement. Optimization works to reduce wirelength by reducing half parameter wirelength
  2. Detailed Placement - Legalizes placement of cells into standard cell rows while adhering to global placement

To do placement in OpenLANE:
```
cd OpenLane
sudo make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design <file_name>
run_synthesis
run_floorplan
run_placement
```

### Viewing Placement in Magic

To view placement in Magic the command mirrors viewing floorplanning:
```
cd ../OpenLane/designs/picorv32a/runs/<most_recent_run>/results/placement/
magic -T ../git_open_pdks/sky130/magic/sky130.tech lef read ../OpenLane/designs/picorv32a/runs/<most_recent_run>/tmp/merged.nom.lef def read picorv32.def &
```

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/72e0c7ed-e596-4ef3-a55f-f5f4cf2537dc)

press 's' and then 'v' to align the design to the center of the screen.

Click on the mouse and press 'z' to zoom into a desired part.

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/c1bad6ce-de18-4dc3-963f-17df36db14b8)

### Standard Cell Design Flow

Cell design is done in 3 parts:

  1. Inputs - PDKs (Process design kits), DRC & LVS rules, SPICE models, library & user-defined specs.
  2. Design Steps - Design steps of cell design involves Circuit Design, Layout Design, Characterization. The software GUNA used for characterization. The characterization can be classified as Timing characterization, Power characterization and Noise characterization.
  3. Outputs - Outputs of the Design are CDL (Circuit Description Language), GDSII, LEF, extracted Spice netlist (.cir), timing, noise, power.libs, function.

### Standard Cell Characterization

Standard Cell Libraries consist of cells with different functionality/drive strengths. These cells need to be characterized by liberty files to be used by synthesis tools to determine optimal circuit arrangement. The open-source software GUNA is used for characterization.

Characterization is a well-defined flow consisting of the following steps:

  1. Link Model File of CMOS containing property definitions
  2. Specify process corner(s) for the cell to be characterized
  3. Specify cell delay and slew thresholds percentages
  4. Specify timing and power tables
  5. Read the parasitic extracted netlist
  6. Apply input or stimulus
  7. Provide necessary simulation commands

 ### General Timing characterization parameters

 #### Timing threshold definitions

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/b237b3ba-2a51-4e65-917b-94f6b463815d)

Propagation Delay The time difference between when the transitional input reaches 50% of its final value and when the output reaches 50% of its final value.
```
Propagation delay=time(out_fall_thr)-time(in_rise_thr)
```
Transition Time The time it takes the signal to move between states is the transition time , where the time is measured between 10% and 90% or 20% to 80% of the signal levels.
```
Rise transition time = time(slew_high_rise_thr) - time (slew_low_rise_thr)
Fall transition time = time(slew_high_fall_thr) - time (slew_low_fall_thr)
```
</details>

<!-- Day 3 Design Library Cell -->
<details>
<summary>Day 3 -- Design Library Cell</summary>

OpenLANE has the benefit of allowing changes to internal switches of the ASIC design flow on the fly. This allows users to experiment with floorplanning and placement without having to reinvoke the tool.

### Spice Simulations

To simulate standard cells spice deck wrappers will need to be created around our model files. 

SPICE deck will comprise of:

  - Model include statements
  - Component connectivity, including substrate taps
  - Output load capacitance
  - Component values
  - Node names
  - Simulation commands
```
*** MODEL DESCRIPTIONS ***
*** NETLIST DESCRIPTION ***
M1 out in vdd vdd pmos W=0.375u L=0.25u
M2 out in 0 0 nmos W=0.375u L=0.25u

cload out 0 10f

Vdd vdd 0 2.5
Vin in 0 2.5
*** SIMULATION Commands ***

.op
.dc Vin 0 2.5 0.05
*** include tsmc_025um_model.mod ***
.LIB "tsmc_025um_models.mod" CMOS_MODELS
.end
```
To plot the output waveform of the spice deck we will use ngspice. The steps to run the simulation on ngpice are as follows:

  1. Source the .cir spice deck file
  2. Run the spice file by: run
  3. Run: setplot → allows you to view any plots possible from the simulations specified in the spice deck
  4. Select the simulation desired by entering the simulation name in the terminal
  5. Run: display to see nodes available for plotting
  6. Run: plot <node> vs <node> to obtain output waveform
```
cd <folder where the .cir file is present>
source CMOS_INVERTER.cir
run
setplot
dc1
display
plot out vs in
```

### Switching Threshold of a CMOS Inverter 

CMOS cells have three modes of operation:

  - Cutoff - No inversion
  - Triode - Inversion but no pinchoff in channel
  - Saturation - Inversion and pinchoff in channel

The voltages at which the switch between the modes of operation happens is dependent on the threshold voltage of the device(s). Threshold voltage is a function of the W/L ratio of a device, therefore varying the W/L ratio will vary the output waveform of CMOS devices. 

To enable efficient description of the varying waveforms a single parameter called switching threshold is used. Switching threshold is defined at the intersection of Vin = Vout. A perfectly symmetrical device will have a switching threshold such that Vin = Vout = VDD/2. 

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/6c0914aa-c33e-4f59-9e47-ce2ff0ee30c7)

### 16 Mask CMOS Process Steps

  - Substrate Selection : Selection of base layer on which other regions will be formed.
  - Create an active region for transistors : SiO2 and Si3N2 deposited. Pockets created using photoresist and lithography.
  - Nwell & Pwell formation : Pwell uses boron and nwell uses phosphorus. Drive in diffusion by placing it in a high temperature furnace.
  - Creating Gate terminal : For desired threshold value NA (doping Concentration) and Cox to be set.
  - Lightly Doped Drain (LDD) formation : LDD done to avoid hot electron effect and short channel effect.
  - Source and Drain formation : Forming the source and drain.
  - Contacts & local interconnect Creation : SiO2 removed using HF etch. Titanium deposited using sputtering.
  - Higher Level metal layer formation : Upper layers of metals deposited.
    
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/19741afd-eb75-48a4-89c7-1eee413b8719)


### Magic Layout View of Inverter Standard Cell

Git clone : https://github.com/nickson-jose/vsdstdcelldesign for cell files.

To invoke Magic:

```
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
magic -T sky130A.tech sky130_inv.mag
```

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/1ef3d7b4-32e3-402d-b1f8-e85bcf14cf89)


### Magic Key Features:

  1. Color Palette - Defines layers and associated colors
Continuous DRC
  2. Device Inference - Automatic recognition of NMOS and PMOS devices

### Device Inference

Select the specific layer/device by hovering over the object and pressing, s, iteratively, until you traverse the hierarchy to the specified object:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/68cf1e13-7196-4b9e-9ed6-164ee442e142)


Run the what command in the tkcon window:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/c839da23-e2a9-4a3c-9481-2c4cb663d337)


### DRC Errors

To check for DRC Errors, select a region (left click for starting point, right click at end point) and see the DRC column at the top that shows how many DRC errors are present.The associated DRC error will be displayed in the tkcon window:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/69a0166f-5d59-48a3-8365-234d60e81e13)

For more information on DRC errors plase refer to: https://skywater-pdk--136.org.readthedocs.build/en/136/
For more information on how to fix these DRC errors using Magic please refer to: http://opencircuitdesign.com/magic/

### PEX Extraction with Magic

To extract the parasitic spice file for the associated layout one needs to create an extraction file; After generating the extracted file we need to output the .ext file to a spice file:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/4a279659-2105-4b93-81eb-fcd01b1315e6)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/4f55e41e-025f-419a-b8ef-0b960faf2e49)


### Spice Wrapper for Simulation

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/b7fbb067-a835-4b9e-b14d-c14c0c96d4d6)

To run the simulation with ngspice, invoke the ngspice tool with the spice file as input:
```
ngspice sky130_inv.spice
plot y vs time a
```
The plot can be viewed by plotting the output vs time while sweeping the input:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/21af853d-4628-4bdd-864b-1ae5949146ab)

</details>
<!-- Day 4 Layout Timing Analysis and CTS -->

<details>
<summary> Day 4 -- Layout Timing Analysis and CTS</summary>

Place and routing (PnR) is performed using an abstract view of the GDS files generated by Magic. The abstract information will include metal and pin information. The PnR tool will use the abstract view information, formally defined as LEF information, to perform interconnect routing in conjunction to routing guides generated from the PnR flow.

### An Introduction to LEF Files

  - Technology LEF - Contains layer information, via information, and restricted DRC rules
  - Cell LEF - Abstract information of standard cells

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/395d32e7-f6ab-4d4e-a47a-2e4e375354fb)


Tracks are used during the routing stage, routes can go over the tracks, or metal traces can go over the tracks. What the file is saying is that for the li1 layer the x or horizontal track is at an offset of 0.23 and a pitch of 0.46. The offset is the distance from the origin to the routing track in either the x or y direction. It is half the pitch so that means the tracks are centered around the origin. 

### Standard Cell Pin Placement

On-track standard cell pin placement is essential for DRC free PnR flow. Pins must align with the li1 and met1 preferred routing directions. During standard cell creation this concept must be accounted for. To ensure a cell is aligned with routing grids in Magic we can display a grid on top of the gds file.

To display the grid in magic:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/50c5ef65-8ca3-4ddf-9746-3f567bd5c937)

Viewing the grid we can ensure our pin placement is optimized for PnR flow:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/bff0731a-8fe1-410a-9154-e2ebc1336c2d)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/d8d84a4a-ef93-4d9c-9477-2d6ed8eae01f)

### LEF Generation in Magic

Magic allows users to generate cell LEF information directly from the Magic terminal. To generate the cell LEF file from Magic perform:
save the modified layout (with new grid)
```
save sky130_vsdinv.mag
```
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/b87a9738-c8e1-4c10-84c7-2c40814d5c46)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/839857b8-fffe-4d00-ac1d-4a9379b6d2bb)

 Open the file and extract LEF
 ```
 magic -T sky130A.tch sky130_vsdinv.mag
 %lef write 
 ```
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/d0443fe7-49be-4b4f-8ff5-a11d13d42a39)

Generated cell LEF file:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/090a6108-74e6-4507-9081-19121a379466)


### Including Custom Cells in OpenLANE

In order to include the new cells in OpenLANE we need to do some initial configuration:
  1. Fully characterize new cell with GUNA for specified corners
  2. Include cell level liberty file in top level liberty file
  3. Reconfigure synthesis switches in the config.tcl file:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/61fa6d0c-b559-4709-b04b-3f2abddf5303)

 ![image](https://github.com/spurthimalode/pes_pd/assets/142222859/442e0c26-a1ac-499c-8ec4-1c5cb591e913)


Note: This step will also include any extra LEF files generated for the custom standard cell(s)
Overwrite previous run to include new configuration switches:

  4. Overwrite previous run to include new configuration switches:
  
 ![image](https://github.com/spurthimalode/pes_pd/assets/142222859/d46de607-942c-4181-b574-9165458ef4ad)


  5. Add additional statements to include extra cell LEFs:
  
 ![image](https://github.com/spurthimalode/pes_pd/assets/142222859/08bf330e-f16c-49d6-bf22-60c22d8f0732)


  5. Check synthesis logs to ensure cell has been integrated correctly

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/3bb39142-7c57-462b-8278-2d6ff9cfc57e)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/42e0ef05-5d7c-4e53-845c-1e717abfa522)


### Fixing Slack Violations

In the architecture design phase of VLSI engineering, system specifications are obtained, which include the required frequency of operation. To assess a circuit's timing performance, static timing analysis (STA) tools are employed, with a particular focus on pre-clock tree synthesis STA analysis, specifically concerning setup timing with respect to a launch clock. STA tools provide valuable insights, reporting issues such as worst negative slack (WNS) and total negative slack (TNS), both related to setup timing constraints. Debugging slack violations can be effectively addressed through STA analysis using OpenSTA, an integrated component within the OpenLANE toolchain.

To ensure the correct operation of these tools and complete the design process, two essential steps must be undertaken:

1. Design Configuration Files (.conf): These files contain tool configuration settings specific to the design under consideration. They provide essential parameters for guiding the design process.

2. Synopsys Design Constraint (.sdc) Files: These files adhere to industry-standard constraints and are crucial for conveying design constraints to the tools accurately.

For a design to be considered complete and reliable, it's imperative that the worst negative slack meets or exceeds 0. If the slack falls outside this desired range, several corrective measures can be taken:

1. Review Synthesis Strategy in OpenLANE: Reevaluate the synthesis approach within OpenLANE to optimize the design for improved timing performance.

2. Enable Cell Buffering: Incorporate cell buffering techniques to enhance signal propagation and mitigate timing issues.

3. Perform Manual Cell Replacement with OpenSTA: Utilize the OpenSTA tool to manually replace cells along the path with worst negative slack (WNS) to optimize timing.

4. Optimize Fanout Value with OpenLANE: Adjust the fanout values using OpenLANE tools to optimize signal distribution and alleviate timing constraints.
To invoke OpenSTA with the configuration file:

```
sta pre_sta.conf
```

### Cell Fanout Example:
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/d62413d8-4362-43da-ba79-45fe60319982)


The delay of this cell is large due to a high load capacitance due to high fanout. To fix this problem we can re-run synthesis within OpenLANE after reconfiguring the maximum fanout load value.

### Cell Replacement Example:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/0a767186-4368-44fb-97b9-3d496f24bd86)

To determine what loads our net is driving in OpenSTA we can report net connecitons:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/70dfdb45-a4bb-4759-8ddb-dbe2f32eac25)

To increase the drive strength of our buffer:

```
OpenSTA> replace_cell_29101_sky130_fd_sc_hd_clkbut_2
```

After performing this optimization we can use the write_verilog command of OpenSTA to output the improved netlist for use in the OpenLANE flow:


### Clock Tree Synthesis

After running floorplan and standard cell placement in OpenLANE we are ready to insert our clock tree for sequential elements in our design. Two of the main concerns with generation of the clock tree are:
  
  1. Clock skew - Difference in arrival times of the clock for sequential elements across the design
  2. Delta delay - Skew introduced through capacitive coupling of the clock tree nets

To run clock tree synthesis (CTS) in OpenLANE:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/40f826fb-e727-4038-b44f-e4e883d2984f)


Note: To ensure timing constraints CTS will add buffers throughout the clock tree which will modify our netlist

### Post-CTS STA Analysis

OpenLANE has the OpenROAD application integrated into its flow. The OpenROAD application has OpenSTA integrated into its flow. Therefore, we can perform STA analysis from within OpenLANE by invoking OpenROAD.

To invoke OpenROAD from OpenLANE:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/50eb437a-1923-4ef2-ab1b-de128e3627d6)


In OpenROAD the timing analysis is done by creating a .db database file. This database file is created from the post-cts LEF and DEF files. To generate the .db files within OpenROAD:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/07127a9c-6626-4b31-bf4a-a36e1eddd14f)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/2aef56eb-7d4f-4f00-a5b5-8150a82e6462)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/9f65cb82-f72a-4932-b868-ac2187a60ba5)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/ed4206a9-e9cf-41d8-85be-a67916743220)

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/c0bb7541-cd45-4092-8386-dcd8ff0fdf28)

</details>
<!-- Day 5 Final Steps in RTL to GDSII -->
<details>
<summary> Day 5--Final Steps in RTL to GDSII</summary>

After generating our clock tree network and verifying post routing STA checks we are ready to generate the power distribution network in OpenLANE:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/3b7c0d48-3e19-42bf-a671-eaae24df76e8)


### Power Distribution Network Generation

To generate the PDN in OpenLANE:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/990eefb5-419c-4b2e-a6a4-ad8296581d21)



The PDN feature within OpenLANE will create:
  1. Power ring global to the entire core
  2. Power halo local to any preplaced cells
  3. Power straps to bring power into the center of the chip
  4. Power rails for the standard cells
  
![image](https://github.com/spurthimalode/pes_pd/assets/142222859/eb147246-d5ec-43b7-ae9c-f111bb1b0caa)


Note: The pitch of the metal 1 power rails defines the height of the standard cells

### Global and Detailed Routing

OpenLANE uses TritonRoute as the routing engine for physical implementations of designs. Routing consists of two stages:

  1. Global Routing - Routing guides are generated for interconnects on our netlist defining what layers, and where on the chip each of the nets will be reputed
  2. Detailed Routing - Metal traces are iteratively laid across the routing guides to physically implement the routing guides

### To run routing in OpenLANE:

```
run-routing
```

If DRC errors persist after routing the user has two options:
  1. Re-run routing with higher QoR settings
  2. Manually fix DRC errors specific in tritonRoute.drc file

### SPEF Extraction

After routing has been completed interconnect parasitics can be extracted to perform sign-off post-route STA analysis. The parasitics are extracted into a SPEF file. The SPEF extractor is not included within OpenLANE as of now. 
```
cd ~/Desktop/work/tools/SPEFEXTRACTOR
python3 main.py <path to merged.lef in tmp> <path to def in routing>
```
The SPEF File will be generated in the location where def file is present
</details>
