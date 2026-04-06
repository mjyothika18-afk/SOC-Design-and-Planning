# SOC-Design-and-Planning

<p>Welcome :)
  
  In this repository, I have documented my learnings through this 10-day workshop organised by VSD (VLSI System Design) in collaboration with NASSCOM. This workshop entails the complete RTL-to-GDSII flow of picorv32a.This workshop has been executed on cloud platform.


<b> Day 1: Inception of open-source EDA, OpenLANE and Sky130 PDK </b>

  - Introduction to OpenLANE and Sky130
  
  - Understanding library cells and flop ratio
  
  - RTL synthesis and timing report analysis

<details>
<summary>Click to expand detailed notes</summary>

---
<b>Understanding RTL-to-GDSII flow:</b>

![images](images/RTL-to-GDSII_1.png)


<b>1. RTL Design (Front-End) </b>
   * Written in Verilog / VHDL
   * Describes functionality (not physical structure)

<b>2. Synthesis </b>
   * Tool converts RTL → gate-level netlist
   * Uses standard cell libraries (.lib)
   * Logic optimization
   * Technology mapping
   * Timing optimization
     
<b>3. Floorplanning </b>
   * Define chip layout structure
   * Die size & core area
   * Macro placement (SRAM, IPs)
   * IO pad placement

<b>4. Power Planning </b>
   * Design power distribution network (PDN)
  
<b>5. Placement </b>
   * Place standard cells in rows
   * Steps:
     -Global placement
     -Detailed placement
     -Optimization for timing & congestion
<b>6. Clock Tree Synthesis (CTS) </b>
   * Build clock distribution network
     Goals:
     -Minimize skew
     -Control latency
     -Reduce clock uncertainty

<b>7. Routing </b>
   * Connect all placed cells
     Types:
     -Global routing (rough paths)
     -Detailed routing (exact metal layers)

<b>8. Signoff Checks </b>
  *  STA (Static Timing Analysis)
     Checks setup & hold violations
  * DRC (Design Rule Check)
     Ensures layout follows foundry rules
  * LVS (Layout vs Schematic)
      Matches layout with netlist
  * IR Drop & EM Analysis
      Checks power integrity

<b>9.  GDSII Generation </b>
  * Final layout file sent to fabrication

---

 <h3>Open source EDA tools:</h3>
 <h3>OpenLane:</h3>
 OpenLane is an open-source RTL-to-GDSII flow used for digital ASIC design. It automates the entire Physical Design process using open-    source tools.

OpenLane is not a single tool — it’s a flow integrating multiple tools:
| Tool | Stage |
|-----|------|
| Yosys,ABC           | → Synthesis          |
| OpenROAD            | → Placement          |
| TritonCTS           | → CTS                | 
| TritonRoute,FastRout| → Routing            |
| Magic,Netgen        | → DRC, layout viewing|
| Netgen              | → LVS checks         | 
| KLayout,Magic       | → GDS visualization  |
| OpenRCX             | → SPEF Extraction    |
| OpenSTA             | → Timing Analysis    |

___

<b> Lab Activities </b>

Invoke the OpenLane tool by navigating to >>Desktop>>OpenLane

```
>cd /home/vscode/Desktop/OpenLane
>make mount
>./flow.tcl -interactive
>package require openlane 1.0.2
```
![image](images/Lab1_invoke.png)
  
```
prep -design picorv32a
```
This command is used as the design setup to set the environment variables,informing the tool about the pdk,standard cell libraries,and also creating the runs directory.

![image](images/prep_design.png)

After we prep the design we are ready to synthesize the design and we use the command run_synthesis.
```
run_synthesis
```

![image](images/run_synthesis.png)

Computing the flop ratio after synthesis:

![image](images/Lab1_flopratio1.png)
![image](images/Lab1_flop_ratio2.png)
```
Flop ratio= (Number of D-ffs)/(Total cell count)
          = 1613/15762
          = 0.10233 (10.23% of the total cell count are d-flip flops)
```
</details>

<b> Day 2: Floorplanning and Physical Preparation </b>


- Introduction to OpenLANE and Sky130  

- Understanding library cells and flop ratio
  
- RTL synthesis and timing report analysis  

<details>
<summary>Click to expand detailed notes</summary>

---

<b>Floorplanning</b> is the first step in Physical Design (PnR) where you define the chip’s physical layout structure before placing standard cells.

1.Die & Core Area Definition
  Die area → total chip size
  Core area → region where standard cells are placed

2.Macro Placement:This is done to minimize routing congestion and to Keep related macros close
  Large blocks like: SRAMs,IPs

3.IO Pad Placement: Input/Output pins placed on chip boundary

4.Aspect Ratio Selection and Utilization: Ratio of height to width of the core
  % of core area used by standard cells is utilization

5.Keep Margins & Blockages
  Keep-out regions → no placement near macros
  Routing blockages → restrict routing areas

  <b>Physical only cells:</b>
  
1.<b>Decap Cells</b> Decoupling capacitors are another type of physical only cells used in PD flow. These do not have any logical functionality.   These cells essentially act as a capacitance between power and ground rails, and hence as a charge reservoir that can be counted upon     while there is a high demand for current from the power lines.


2.<b>Endcap Cells</b>: Placed at the edges of standard cell rows or blocks to terminate diffusion/well regions, prevent DRC violations, and        provide structural integrity.


3.<b>Tap Cells</b>: Connect substrate and well regions to power rails, preventing floating wells, stabilizing body bias, and avoiding latch-up     conditions.

---
<b>Lab activities:</b>

Let's start our PD flow with floorplan.

Info:Since I am starting floorplan separately, I will need to start with the previous steps again i.e from make mount until run_synthesis(also since prep -design <design_name> usually creates a run directory each time I run the command we can include a switch called -tag (provide the name of your previous run file) -overwrite.(just to avoid creation of multiple run directories)

```
run_floorplan
```

![image](images/floorplan.png)

Now we can visialize the DEF file generated after floorplan in Magic

```
magic –T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech lef read ../../merged.nom.lef def read picorv32a.def &
```

![image](images/floorplan_magic.png)

We can user 'S' to select the design and type 'V' to bring the design to the centre.

Once we centre the design move the cursor to the element of your interest and type 'S' to know the element's name use the tkcon window and type 'what'

![image](images/floorplan_2_in_magic.png)

![image](images/viewing_select_in_magic.png)

![image](images/viewing_what_in_magic_floorplan.png)


Now,we can use run_placement command to place the standard cells in our design.
```
run_placement
```

![image](images/placement.png)
</details>


<b> Day 3: Design and Characterisation of Library Cells using Magic & ngspice </b>


- CMOS Inverter — SPICE Deck,Spice extraction of inverter in magic.Editing the spice model file for analysis through simulation andPost-    layout ngspice simulations.

- 16-Mask CMOS Fabrication Process
  
- Find problem in the DRC section of the old magic tech file for the skywater process and fix them.  

<details>
<summary>Click to expand detailed notes</summary>

---

CMOS Fabrication process: 

A 16-mask CMOS fabrication process uses multiple lithography steps to sequentially form wells, isolation, gates, source/drain regions, contacts, and multi-layer metal interconnects, resulting in a complete integrated circuit.

```

Substrate
   ↓
Well Formation (N/P)
   ↓
Isolation (STI)
   ↓
Gate Formation
   ↓
Source/Drain
   ↓
Contacts
   ↓
Metal Layers
   ↓
Passivation
   ↓
Pad Openings
   ↓
Final Chip

```

Layout view VS Abstract view

1. Layout View

Layout = Actual physical implementation

Contains:
Exact geometry (polygons)
Metal layers, vias
Diffusion, poly
Used for:
DRC
LVS
Final GDSII

2. Abstract View

Abstract = Simplified representation of layout

Contains:
Block boundaries
Pin locations
Routing blockages
Does NOT include:
Full geometry details

Used for:

Placement
Floorplanning
Routing estimation

<b>CMOS Inverter simulation parameters:</b>

1.W/L Ratios (Transistor Sizing): Width (W) and Length (L) of PMOS and NMOS transistors, critical for balancing pull-up and pull-down times (setting rise/fall times).

2.Propagation Delay (tp): The time delay between input and output changes, calculated by averaging high-to-low (tphl) and low-to-high (tplh) delays (measured at 50% points).

3.Rise and Fall Times (tr,tf): Time taken for the output to transition between 10% and 90% of voltage levels.

4.Noise Margins (NMh,NMl):The ability to withstand noise on input without affecting output (NMl=VIL-VOL).(VIL-Input Low, VOL-Output Low)

---

<b>Lab activities</b>

Cloning and Characterising a Custom Inverter Cell

```
#First clone the directory
git clone https://github.com/nickson-jose/vsdstdcelldesign.git

#Copy the tech file to vsdstdcell dirctory created inside OpenLane upon cloning.Then execute the below command.
magic -T sky130A.tech sky130_inv.mag &
```

![image](images/magic_inverter_1.png)

![image](images/magic_inverter_2.png)

![image](images/magic_inverter_3.png)

![image](images/magic_inverter_7_extract.png)

Extracting SPICE Netlist from Magic

```
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```
![image](images/ext_spice_generated.png)

![image](images/ext_spice_updated.png)

![image](images/magic_inverter_5_bbox.png)

![image](images/ext_spice_updated_2.png)

![image](images/ng_spice_inv.png)

![image](images/ng_spice_inv_plot.png)

![image](images/Rise1.png)

![image](images/Rise2.png)

![image](images/rise_xy.png)

![image](images/fall1.png)

![image](images/fall2.png)

![image](images/fall3.png)

![image](images/Propagation_delay1.png)

![image](images/Propagation_delay2_output.png)

![image](images/Propagation_delay_calculation.png)


DRC rules: poly.9 rule correction

![image](images/drc_rules.png)

![image](images/drc1.png)

![image](images/drc2.png)

![image](images/drc3.png)

![image](images/drc4.png)


