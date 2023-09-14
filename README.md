# Advanced Physical Design Using OPENLANE/SKY130
<details>
  <summary>Prerequisites</summary>
  
 1. Ubuntu OS-based System
  
 2. 25GB+ Disk Space
      
 Please refer to: https://github.com/nickson-jose/openlane_build_script for installation steps
</details>
<details>
<summary>RTL to GDSII Introduction</summary>

From conception to product, the ASIC design flow is an iterative process that is not static for every design. The details of the flow may change depending on ECO’s, IP requirements, DFT insertion, and SDC constraints, however the base concepts still remain. The flow can be broken down into 11 steps:

  1. Architectural Design – A system engineer will provide the VLSI engineer with specifications for the system that are determined through physical constraints. The VLSI engineer will be required to design a circuit that meets these constraints at a microarchitecture modeling level.

  2. RTL Design/Behavioral Modeling – RTL design and behavioral modeling are performed with a hardware description language (HDL). EDA tools will use the HDL to perform mapping of higher-level components to the transistor level needed for physical implementation. HDL modeling is normally performed using either Verilog or VHDL. One of two design methods may be employed while creating the HDL of a microarchitecture:

      a. 	RTL Design – Stands for Register Transfer Level. It provides an abstraction of the digital   circuit using:
      
      <ul>
        <li>i. 	Combinational logic</li>
        <li>ii. 	Registers</li>
        <li>iii. 	Modules (IP’s or Soft Macros)</li>
      </ul>

      b. 	Behavioral Modeling – Allows the microarchitecture modeling to be performed with behavior-based modeling in HDL. This method bridges the gap between C and HDL allowing HDL design to be performed

  3. RTL Verification - Behavioral verification of design

  4. DFT Insertion - Design-for-Test Circuit Insertion

  5. Logic Synthesis – Logic synthesis uses the RTL netlist to perform HDL technology mapping. The synthesis process is normally performed in two major steps:

  <ul>
      <li> GTECH Mapping – Consists of mapping the HDL netlist to generic gates what are used to perform logical optimization based on AIGERs and other topologies created from the generic mapped netlist.</li>
      <li>Technology Mapping – Consists of mapping the post-optimized GTECH netlist to standard cells described in the PDK</li>
  </ul>
        
Standard Cells – Standard cells are fixed height and a multiple of unit size width. This width is an integer multiple of the SITE size or the PR boundary. Each standard cell comes with SPICE, HDL, liberty, layout (detailed and abstract) files used by different tools at different stages in the RTL2GDS flow.

  6. Post-Synthesis STA Analysis: Performs setup analysis on different path groups.

  7. Floorplanning – Goal is to plan the silicon area and create a robust power distribution network (PDN) to power each of the individual components of the synthesized netlist. In addition, macro placement and blockages must be defined before placement occurs to ensure a legalized GDS file. In power planning we create the ring which is connected to the pads which brings power around the edges of the chip. We also include power straps to bring power to the middle of the chip using higher metal layers which reduces IR drop and electro-migration problem.

  8. Placement – Place the standard cells on the floorplane rows, aligned with sites defined in the technology lef file. Placement is done in two steps: Global and Detailed. In Global placement tries to find optimal position for all cells but they may be overlapping and not aligned to rows, detailed placement takes the global placement and legalizes all of the placements trying to adhere to what the global placement wants.

  9. CTS – Clock tree synteshsis is used to create the clock distribution network that is used to deliver the clock to all sequential elements. The main goal is to create a network with minimal skew across the chip. H-trees are a common network topology that is used to achieve this goal.

  10.  Routing – Implements the interconnect system between standard cells using the remaining available metal layers after CTS and PDN generation. The routing is performed on routing grids to ensure minimal DRC errors.
    
The Skywater 130nm PDK uses 6 metal layers to perform CTS, PDN generation, and interconnect routing.
Shown below is an example of a base RTL to GDS flow in ASIC design:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/1de4339c-7d98-41ff-812b-b7531bf73d64)

</details>

<!-- Workshop Introduction -->
## Workshop Introduction

The inputs to the ASIC design flow are:

    - Process Design Rules: DRC, LVS, PEX
    - Device Models (SPICE)
    - Digital Standard Cell Libraries
    - I/O Libraries

Process Design Kit (PDK) is the interface between the CAD designers and the foundry. The PDK is a collection of files used to model a fabrication process for the EDA tools used in designing an IC. PDK’s are traditionally closed-source and hence are the limiting factor to open-source Digital ASIC Design. Google and Skywater have broken this stigma and published the world’s first open-source PDK on June 30th, 2020. This breakthrough has been a catalyst for open-source EDA tools. This workshop focuses on using the open-source RTL2GDS EDA tool, OpenLANE, in conjunction with the Skywater 130nm PDK to perform the full RTL2GDS flow as shown below:

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

Two key descriptions of a floorplan are utilization and aspect ratio. The amount of area of the die core the standard cells are taking up is called utilization. Normally we go for 50-70% utilization to, or utilization factor of 0.5-0.7. Keeping within this range allows for optimization of placement and realizable routing of a system. Aspect ratio can specify the shape of your chip by the height of the core area divided by the width of the core area. An aspect ratio of 1 discribes the chip as a square.

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
run_placement
```

### Viewing Placement in Magic

To view placement in Magic the command mirrors viewing floorplanning:

![image](https://github.com/spurthimalode/pes_pd/assets/142222859/72e0c7ed-e596-4ef3-a55f-f5f4cf2537dc)


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
</details?
